---
title: Diving in Mariana Trench (episode 2)
author: Paolo Serra
date: 2023-02-12 10:00:00
categories: [Mobile Security, Mariana Trench]
toc: true
author: paoloserra
img_path: /images/mariana-trench/part_2/
image:
  path: wallpaper.jpeg
  width: 100%
  height: 315
---

Episode of [Mariana Trench](/categories/mariana-trench/) season 1.


## A Lot Came Out
Welcome to the second episode. After giving you a panoramic view of what Mariana Trench is, you should have sufficient knowledge to level up and create something slightly more sophisticated now. It is crucial to consolidate the key concepts around the behavior and features of Mariana to build efficient models that allow you to discover in-depth vulnerabilities. The further we go, the more complicated models you will see.

In this episode, we will enhance our models by adding other helpful constraints to better identify the sources and sinks based on the goals we want to achieve. The idea behind Mariana Trench was to create a static analyzer aimed at discovering all possible data flows that may pose security risks regardless of generating a high number of false positives ([cit](https://engineering.fb.com/2021/09/29/security/mariana-trench/#:~:text=In%20using%20MT%20at%20Facebook%2C%20we%20prioritize%20finding%20more%20potential%20issues%2C%20even%20if%20it%20means%20showing%20more%20false%20positives.%20This%20is%20because%20we%20care%20about%20edge%20cases%3A%20data%20flows%20that%20are%20theoretically%20possible%20and%20exploitable%20but%20rarely%20happen%20in%20production.)). 
Being the tool highly configurable, it provides you with a customizable generation of models and rules you are only interested in, which might generate a reasonable number of false positives. However, it may become a mixed blessing if you stay too general when defining the models.
- the more general the models are, the more false positives you may get
- the more precise the models are, the fewer vulnerabilities you may get

## White Box vs Black Box
Do you remember when I mentioned the workarounds for dealing with the lack of a codebase? I kept the promise, and here we are. Mariana Trench is supposed to work with an APK and its source code, even though it does the job without a codebase. That’s because the code analyzed is the Dalvik bytecode, not the Java classes. The source code is especially needed in the UI when reviewing the issues found, thus showing the parts of the code where the data flows.

You don't have the source code in black-box, but there are many tools that can extract the Java code from an APK. One is [***jadx-gui***](https://github.com/skylot/jadx), which also allows you to save the output as a gradle project:

![Window shadow](jadx-save-as-grandle.png){:  width="1548" height="864"  }
_jadx-gui_

Jadx's output is not enough to work with SAPP UI since, when given the sources, it displays the traces with 404 HTTP errors:
![Window shadow](mariana-trench-error-not-found.png){:  width="1548" height="864"  }
_File Not Found_

We notice an error indicating that a path file was not found by inspecting the responses in the console:
![Window shadow](console-error.png){: .shadow   }
_GraphQL API_

Looking inside that path, you can notice the file has the extension `.java`{: .filepath}, and that’s why the error occurs. Jadx saves the sources with the extension, so we need to rename the files without it. Here is the command to rename all sources:

```console
$ find <PATH_JADX_OUTPUT>/app/src/main/java -type f -name '*.java' -exec sh -c 'mv -- "$1" "${1%.java}"' sh {} \;
```

Once all the files are renamed, SAPP UI displays the traces correctly:
![Window shadow](trace-file-found.png){:    }
_/tmp/apps/app-debug/app/src/main/java/com/mariana_lab/MainActivity_

> Limitations: other errors may occur, for example a mismatch between the lines. Furthermore, the highlighted line is often incorrect when using these workarounds, so do not rely on it.
{: .prompt-warning }

There are five steps between decompiling the APK and starting SAPP UI, with respect to my laziness, I created a [bash script](https://github.com/serrapa/mariana-trench-launcher) runnable inside the Mariana Trench directory: 

```console
$ cd <MARIANA_PATH> & source venv/bin/activate & ./mariana-.sh <APK_PATH> [dev] [skip]
```



## Get your hands dirty - again
In the previous post, I introduced you to the basic concepts of Mariana Trench through a practical example. We developed two simple models: one source and one sink. However, they were too general, *a piece of cake*. Building simple generators might generate a large volume of false positives and, therefore, we will do our best to improve our models.


### Challenge 1
Let’s get back to our [Ovaa](https://github.com/oversecured/ovaa) application and custom files for Mariana Trench. In this episode, we will tune our model generators to be more efficient and minimize the false positives we might get with the current files. Before going on, a short recap ([docs](https://mariana-tren.ch/docs/models/)) of the main concepts:

- **Model generator**: A model is an abstract representation of how data flows through a method and it can consist of sources, sinks, and other things. Our `CustomDeeplinkDataSourceGenerator`{: .filepath} and `CustomSharedPreferencesSinkGenerator`{: .filepath}  are the model generator files where we defined the conditions to declare what methods should be considered a source or a sink.

- **Constraint**: an expression that specifies the methods for which a “model” should be generated. A method or field must satisfy all constraints in order to generate a model for it.

Once all the models are generated, Mariana does its job and produces a result that is not human-readable and needs to be analyzed by the [SAPP](https://github.com/facebook/sapp) PyPi tool.

#### Minimize the false positives
Basically, we designed two models based on the ***name*** constraint, which treats the method as a source only if the item contains the specified value. It doesn't care about the class, the arguments' types, or the return value. Each method starting with the ```getQueryParameter```  or the ```putString``` string is treated as a source. 

| Source/Sink                                                                                     | Item                                    |
|:--------------------------------------------------------------------------|:-----------------------------------------|
|![Window shadow](check_icon.png){: w="22" h="22" } | Lcom/example/classA;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;| 
|![Window shadow](check_icon.png){: w="22" h="22" } | Lcom/example/classA;.**putStringIntoRequest**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;| 
|![Window shadow](check_icon.png){: w="22" h="22" } | Lcom/example/classA;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Lcom/example/classB;| 
|![Window shadow](check_icon.png){: w="22" h="22" } | Landroid/content/SharedPreferences$Editor;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;|

Let's try to be more efficient. When you define a constraint, you have more options:
- *signature* : specify a regex to fully match the full signature of the target method.
- *name* : specify a regex to fully match the name of the target method.
- *parent* : specify a nested constraint to apply to the class holding the target method.

In the previous episode, we used only the *name* without specifing any constraints about the class of the method we were interested in. Since our focus is on the methods belonging to the `Uri` class and `Editor` interface, we need to add a *parent* constraint to our code to minimize false positives:

````json
{
  "model_generators": [
    {
      "find" : "methods",
      "where" : [
        {
          "constraint": "parent",
          "inner": {
            "constraint": "name",
            "pattern": "Landroid/net/Uri;"
          }
        },
        {
          "constraint": "any_of",
          "inners" : [
            { "constraint": "name", "pattern": "getQuery" },
            { "constraint": "name", "pattern": "getQueryParameter.*" }
          ]
        }
      ],
      "model" : {
        "modes": [
          "skip-analysis"
        ],
        "sources": [
          {
            "kind": "DeeplinkUserInput",
            "port": "Return"
          }
        ]
      },
      "verbosity" : 1
  }]
}
````
{: file="DeepLink Source - CustomDeeplinkDataSourceGenerator.json" }

````json
{
  "model_generators": [
    {
      "find" : "methods",
      "where" : [
        {
          "constraint": "parent",
          "inner": {
            "constraint": "name",
            "pattern": "Landroid/content/SharedPreferences\\$Editor;"
          }
        },
        {
          "constraint": "any_of",
          "inners" : [
            { "constraint": "name", "pattern": "putString" },
            { "constraint": "name", "pattern": "putStringSet" },
            { "constraint": "name", "pattern": "putLong" },
            { "constraint": "name", "pattern": "putInt" },
            { "constraint": "name", "pattern": "putFloat" },
            { "constraint": "name", "pattern": "putBoolean" }
          ]
        }
      ],
      "model" : {
        "modes": [
          "skip-analysis"
        ],
        "for_all_parameters": [
          {
            "variable": "x",
            "sinks": [
              {
                "kind": "writeOnSharedPreferences",
                "port": "Argument(x)"
              }
            ]
          }]
      },
      "verbosity" : 1
  }]
}
````
{: file="SharedPreferences Sink - CustomSharedPreferencesSinkGenerator.json" }

The JSON definition of our model has been increased a bit. Let me give you some explanations:
- The `parent`  constraint needs another key-value pair: the key is **inner**, and the value is a nested constraint. Remember that the **pattern** keyword specifies a regex, and classes take the form `Lpackage/name/ClassName;`{: .filepath} (you can find it on the Android Doc). Editor, on the other hand, is an interface so you must prepend the `\\$`{: .filepath} string to the name. Double escaping is required for regex and json syntax.

- We defined an `any_of` constraint that takes a list of constraints. Its name explains itself.

- A trick to make Mariana Trench's analysis faster and more intelligent is to instruct it to decide whether or not to do the analysis of the method(s) that satisfies the constraint. Since we know all the methods defined in the *any_of* constraint are provided by the Android APIs, we don't need to perform an analysis there as well. The `skip-analysis` mode does it. The documentation says this mode is set by default if there's no source code for the method.

- Every time we set constraints on multiple parameter methods and don't know where the position of our sink parameter is, we can use `for_all_parameters` to define variables.

After tuning, we can say we achieved our goal of minimizing false positives. We are now sure that we have defined a model that handles the methods held by the proper classes.

| Source/Sink                                                                                     | Item                                    |
|:--------------------------------------------------------------------------|:-----------------------------------------|
|![Window shadow](uncheck_icon.png){: w="22" h="22" }| Lcom/example/classA;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;| 
|![Window shadow](uncheck_icon.png){: w="22" h="22" }| Lcom/example/classA;.**putStringIntoRequest**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;| 
|![Window shadow](uncheck_icon.png){: w="22" h="22" }| Lcom/example/classA;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Lcom/example/classB;| 
|![Window shadow](check_icon.png){: w="22" h="22" }| Landroid/content/SharedPreferences$Editor;.**putString**:(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;|

##### Spoiler
In the next episode, we will focus on the SAPP UI to better understand what we can achieve along with it that cannot be with Mariana itself. Additionally, we will backtrack on [Step 2](/posts/mariana_trench/#step-2) and [Step 3](/posts/mariana_trench/#step-3) to generate the necessary rules to complete the first challenge of the Ovaa application with minimal effort and without having to go deeper into the source code to spot the issue.

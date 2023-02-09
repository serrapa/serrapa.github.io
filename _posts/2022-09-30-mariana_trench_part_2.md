---
title: Diving in Mariana Trench (part 2)
author: Paolo Serra
date: 9999-09-07 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mariana-trench/part_2/
image:
  path: wallpaper.jpeg
  width: 100%
  height: 315
---

Episode of [Mariana Trench](/posts/mariana_trench/) season.


## From shell to hell
Welcome to the second episode. 

## White Box vs Black Box
Do you remember I mentioned the workarounds for dealing with the lack of a codebase? I kept the promise, and here we are. Mariana Trench is supposed to work with an APK and its source code, even though it does the job without a codebase. That’s because the code analyzed is the Dalvik bytecode, not the Java classes. The source code is especially needed in the UI when reviewing the issues found, thus showing the parts of the code where the data flows.

In black-box, you don’t have the source code, but there are many tools to get back the Java code from an APK. One is [***jadx-gui***](https://github.com/skylot/jadx), which also allows you to save the output as a gradle project:

![Window shadow](jadx-save-as-grandle.png){: .shadow width="1548" height="864" style="max-width: 90%" }
_jadx-gui_

Jadx's output is not enough to work with SAPP UI since, when given the sources, it displays the traces with 404 HTTP errors:
![Window shadow](mariana-trench-error-not-found.png){: .shadow width="1548" height="864" style="max-width: 90%" }
_File Not Found_

We may see an error indicating that a path file was not found by inspecting the responses in the console:![Window shadow](console-error.png){: .shadow  style="max-width: 90%" }
_GraphQL API_

Looking inside that path, you can notice the file has the extension `.java`{: .filepath}, and that’s why the error occurs. Jadx saves the sources with the extension, so we need to rename the files without it. Here is the command to rename all sources:

```console
$ find <PATH_JADX_OUTPUT>/app/src/main/java -type f -name '*.java' -exec sh -c 'mv -- "$1" "${1%.java}"' sh {} \;
```

Once all the files are renamed, SAPP UI displays the traces correctly:
![Window shadow](trace-file-found.png){: .shadow  style="max-width: 90%" }
_/tmp/apps/app-debug/app/src/main/java/com/mariana_lab/MainActivity_

> Limitations: other errors may occur, indicating a mismatch between the lines or for other causes. Furthermore, the highlighted line is often incorrect when using these workarounds, so do not rely on it.
{: .prompt-warning }

There are five steps between decompiling the APK and starting SAPP UI, and since we are lazy, I made a [bash script](https://github.com/serrapa/mariana-trench-launcher) to run inside the Mariana Trench directory: 

```console
$ cd <MARIANA_PATH> & source venv/bin/activate & ./mariana-.sh <APK_PATH> [dev] [skip]
```



## Get your hands dirty - again
In the previous post, I showed you a practical example to introduce the basic concepts of Mariana Trench. We developed two simple models, one for the source and the other for the sink, but there was a problem: they were *a piece of cake*. Building simple generators might generate a large volume of false positives. 

Our goal in this episode are:
- complete the challenge
- improve our model generators


### Challenge 1
Let's get back on our [Ovaa](https://github.com/oversecured/ovaa) application and our custom files for mariana trench. 

#### Minimize the false positives
Basically, we designed two models based on the ***name*** constraint, which treats the method as a source only if the item contains the specified value. It doesn't care about the parent method, the arguments, the types and the return value. Each method with the ```getQueryParameter```  or the ```putString``` substring it's considered as a source. 

Let's try to be more efficient. When you define a constraint, you have more options:
- *signature* : specify a regex to fully match the full signature of the target method.
- *name* : specify a regex to fully match the name of the target method.
- *parent* : specify a nested constraint to apply to the class holding the target method.

In the previous episode, we used only the *name*, without specifing any constraint about the class of the method we are interested in. Since our attention is directed towards the methods belonging to the `Uri` class, we need to add a *parent* constraint to our code to minimize false positives:

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
- The `parent`  constraint needs another key-value pair: the key is **inner**, and the value is a nested constraint. Remember that the **pattern** keyword specifies a regex, and classes take the form `Lpackage/name/ClassName;`{: .filepath} (you find it on the Android Doc). In the case of `CustomSharedPreferencesSinkGenerator.json`{: .filepath}, Editor is an interface, though thus you must prepend the `\\$`{: .filepath} string to the name. Double escaping is required for regex and json syntax.

- We defined an `any_of` constraint that takes a list of constraints. Its name explains itself.

- A trick to make Mariana Trench's analysis faster and more intelligent is to instruct it to decide whether or not doing the analysis of the method(s) that satisfies the constraint. Since we know all the methods defined in the *any_of* constraint are provided by the Android APIs, we don't need to perform an analysis also there. The `skip-analysis` mode does it.

- Every time we set constraints on multiple parameter methods and don't know where is the position of our sink parameter, we can use `for_all_parameters` to define variables.

After tuning, we can say we completed our goal of minimizing false positives. We are now sure that we are defined a model that handles the methods held by the proper classes.

##### Conclusion
In the next episode, we will see 

- analizzeremo la UI
- minimizzare i falsi positivi usando i filter sulla UI (si può fare concatenando le regole? Forse l'unico modo è con le feature) features in action 
- scriveremo la regola per trovare "a flow where data stored in the shared preferences are processed in an HTTP request" e cercheremo di creare una regola unica per identificare l'insieme dei due flow analizzati: input che proviene dal deeplink salvato nelle shared preferences, poi recuperato e mandato in una richiesta http.

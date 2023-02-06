---
title: Diving in Mariana Trench (part 2)
author: Paolo Serra
date: 9999-09-07 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/
image:
  path: wallpapers/callofthenight.jpeg
---

## From shell to hell
Welcome to the second part of this series. Are you ready to deal with the UI? It is time to move on to the SAPP UI  to navigate the issues tracked. 

## White Box vs Black Box
Do you remember I mentioned the workarounds for dealing with the lack of a codebase? I kept the promise, and here we are. Mariana Trench is supposed to work with an APK and its source code, even though it does the job without a codebase. That’s because the code analyzed is the Dalvik bytecode and not the java classes. The source code is especially needed in the UI when reviewing the issues found, thus showing the parts of the code where the data flows. 

In black-box, you don’t have the source code, but there are many tools to get back the Java code from an APK. My favourite one is [***jadx-gui***](https://github.com/skylot/jadx), which also allows you to save the output as a gradle project:

![Window shadow](mariana-trench/jadx-save-as-grandle.png){: .shadow width="1548" height="864" style="max-width: 90%" }
_jadx-gui_

Jadx's output is not enough to work with SAPP UI since, when given the sources, it displays the traces with 404 HTTP errors:
![Window shadow](mariana-trench/mariana-trench-error-not-found.png){: .shadow width="1548" height="864" style="max-width: 90%" }
_File Not Found_

We may see an error indicating that a path file was not found by inspecting the responses in the console:![Window shadow](mariana-trench/console-error.png){: .shadow  style="max-width: 90%" }
_GraphQL API_

Looking inside that path, you can notice the file has the extension `.java`{: .filepath}, and that’s why the error occurs. Jadx saves the sources with the extension, so we need to rename the files without it. Here is the command to rename all sources:

```console
$ find <PATH_JADX_OUTPUT>/app/src/main/java -type f -name '*.java' -exec sh -c 'mv -- "$1" "${1%.java}"' sh {} \;
```

Once all the files are renamed, SAPP UI displays the traces correctly:
![Window shadow](mariana-trench/trace-file-found.png){: .shadow  style="max-width: 90%" }
_/tmp/apps/app-debug/app/src/main/java/com/mariana_lab/MainActivity_

> Limitations: other errors may occur, indicating a mismatch between the lines or for other causes. Furthermore, the highlighted line is often incorrect when using these workarounds, so do not rely on it.
{: .prompt-warning }

There are five steps between decompiling the APK and starting SAPP UI, and since we are lazy, I made a bash script to run inside the Mariana Trench directory: 

```console
$ cd <MARIANA_PATH> && ./speedmt.sh <APK_PATH>
```

```bash
#!/bin/bash

ORANGE='\033[0;33m'
LIGHT_GREEN='\033[1;32m'
NC='\033[0m' # No Color

APK_PATH=$1
SKIP_DECOMPILATION=$2
APP_NAME=`basename $APK_PATH | cut -f 1 -d '.'`
APK_SOURCE_CODE="/tmp/apps/${APP_NAME}/app/src/main/java"

echo -e "APP NAME: ${ORANGE}${APP_NAME}${NC}"

if [[ -d /tmp/apps/$APP_NAME ]]; then
 echo "The apk's directory already exists. Anything inside will be overwritten!"
else
   echo "The apk's directory does not exists and it will be created."
fi


jadx -q -j 10 -d /tmp/apps/$APP_NAME -e $APK_PATH
if [ $? -eq 0 ]; then
   echo -e "${LIGHT_GREEN}[JADX]${NC} Completed. Output on \"/tmp/apps/${APP_NAME}\""
else
   echo -e "${RED}[JADX]${NC} An issue occurred."
   exit 1
fi


find /tmp/apps/$APP_NAME/app/src/main/java -type f -name '*.java' -exec sh -c 'mv -- "$1" "${1%.java}"' sh {} \;
if [ $? -eq 0 ]; then
   echo -e "${LIGHT_GREEN}[FIND]${NC} Completed."
else
   echo -e "${RED}[FIND]${NC} An issue occurred."
   exit 1
fi

mkdir /tmp/apps/$APP_NAME/output

mariana-trench --system-jar-configuration ~/Library/Android/sdk/platforms/android-30/android.jar \
    --apk-path $APK_PATH \
    --output-directory /tmp/apps/$APP_NAME/output \
    --verbosity 1
if [ $? -eq 0 ]; then
   echo -e "${LIGHT_GREEN}[MARIANA-TRENCH]${NC} Completed. Output on \"/tmp/apps/${APP_NAME}/output\""
else
   echo -e "${RED}[MARIANA-TRENCH]${NC} An issue occurred."
   exit 1
fi

sapp --tool=mariana-trench analyze /tmp/apps/$APP_NAME/output
if [ $? -eq 0 ]; then
   echo -e "${LIGHT_GREEN}[SAPP]${NC} Analysis completed."
else
   echo -e "${RED}[SAPP]${NC} We got an issue during the analysis."
   exit 1
fi

sapp --database-name=sapp.db server --source-directory=$APK_SOURCE_CODE --debug
````

## Step up to the intermediate level: features e propagations
It's time to level up.


## Get your hands dirty - again
In the previous post, I showed you a practical example to introduce the basic concepts of Mariana Trench. We developed two simple models, one for the source and the other for the sink, but there was a problem: they were *a piece of cake*. Building simple generators might generate a large volume of false positives. Basically, we designed two models based on the ***name*** constraint, which treats the method as a source only if the item contains the string defined. It doesn't care about the parent method, the arguments, the types and the return value. Each method with the ```getQueryParameter```  or the ```putString``` string inside it is treated as a source.


### Challenge 1


#### Minimize the false positives
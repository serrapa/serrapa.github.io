---
title: Diving in Mariana Trench (episode 3)
author: Paolo Serra
date: 9999-01-01 10:00:00
categories: [Mobile Security, Mariana Trench]
toc: true
author: paoloserra
img_path: /images/mariana-trench/part_3
image:
  path: wallpaper.jpeg
---

Episode of [Mariana Trench](/categories/mariana-trench/) season 1.


## From shell to hell
The UI

## Step up to the intermediate level: features e propagations
It's time to level up and explore some other features beyond sources and sinks. 



- analizzeremo la UI
- minimizzare i falsi positivi usando i filter sulla UI (si può fare concatenando le regole? Forse l'unico modo è con le feature) features in action 
- scriveremo la regola per trovare "a flow where data stored in the shared preferences are processed in an HTTP request" e cercheremo di creare una regola unica per identificare l'insieme dei due flow analizzati: input che proviene dal deeplink salvato nelle shared preferences, poi recuperato e mandato in una richiesta http.
---
title: Microsoft365 E5 续订——Github Actions
date: 2022-01-27T20:53:30+08:00
draft: true
---

基于 AutoApi v6.4

fork from <https://github.com/Innersider/cactial-AutoAPI.git>

## 基本知识

## v6.4

action 添加结束workflow? <https://github.com/canmeng99/AutoApi>

```yml
build:
    ......
    steps:
        ......
        - name: Delete workflow runs
            uses: GitRML/delete-workflow-runs@main
            with:
            retain_days: 1
            keep_minimum_runs: 1
```

## 添加 workflow 保持action活跃

action keep workflow alive <https://github.com/ccknbc-actions/E5-Developer>

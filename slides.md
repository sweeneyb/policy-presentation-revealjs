---
author: Brian Sweeney
title: Policy For Use-Case Awareness
date: Oct 5, 2023
---

## This is my first slide 

And this is some text

* item 1

::: incremental

* item 2
:::


---

# How did we get here?
* You're a multitenant platform SRE
* You provide a range of features, but powers users want a narrow "happy path"
* How can you let the know when somebody goes off path?

---

## Let's validate email domains
### (the data)
```
name: Second
size: 72
accountFlows:
  - act_flow_001:
    responsibleEmail: vhhcjfjl@sharklasers.com
    FlowNodes:
      - name: Flow_001
        address: flow001.vcap.me
        region: US
        parallelism: 5
      - name: Flow_002
        address: flow002.vcap.me
        region: US
        parallelism: 5
  - act_flow_002:
    responsibleEmail: vhhcjfjj@sharklasers.com
  - act_flow_003:
    responsibleEmail: vhhcjfjj@vcap.me
```


---

## Slide 2

A second slide

```
code block() {
    something();
}

```
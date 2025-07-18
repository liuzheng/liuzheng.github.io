---
layout: post
title: "Airflow for Operation"
tagline: ""
description: ""
category: operation
tags: [operation,airflow]
---

## 前言

在自动化运维领域持续深耕，一直在寻找最优雅的自动化运维方式，本文是基于现有技术探索认为当前最优雅的自动化运维方式，如有更好的建议或意见，欢迎联系。

一直以来，关于自动化运维都是围绕着 Ansible & Salt & Puppet & Chef 等联通能力做运维手段的基石，然后由于各自不统一，上层的封装和使用就变得千奇百怪，典型的 Ansible 衍生出了 Tower/AWX 这个工具产品，并不是说这个工具不健全，只是这些自动化工具的封装方向都具有一定的排他性或者局限性，导致需要多个平台，来共同管理这些运维功能需求。

就工作而言，多个平台意味着多个人来管理，就工作而言，多个平台意味着精力的分散，或者无法深入某个系统，导致使用部分功能和能力，在管理上也会存在一定的缺陷。

我不否认任何一个平台的设计者的能力和功能的完整程度，但是确实对于平台的部署&运维&使用者来说，确实增加了很多门槛。就拿携程内部自动化运维工作平台来说，就无法避免的部署了多套来保障整体的运维覆盖度，这对于集中式运维就很麻烦，可能某些公司使用了蓝鲸或自研的系统，这里不过度聊。仅对于单纯的运维能力提升，并且包含的是运维人员的权力封装及分发给用户这个领域，我希望做的纯粹一点。

## 工具选型
我是一个 K8S 的簇拥，但是又是一个纯粹的追求者，K8S 在我看来还是属于应用，依然需要运维手段来管理 Node。初期选型的时候我第一支持的就是 Argo，但在我领导的建议下还是选择了Airflow（他也是纯粹的追求者），无外乎运维隔离，这个确实是无法逃避的选项，还记得早年在SAP的时候还是必须搭建 Rancher 来管理 K8S 一样，工具手段的隔离。


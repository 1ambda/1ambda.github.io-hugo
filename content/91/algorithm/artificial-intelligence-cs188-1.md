+++
date = "2016-06-25T13:25:02+09:00"
next = "../artificial-intelligence-cs188-2"
prev = "../"
title = "AI (CS188): Intro"
toc = true
weight = 51
aliases = [
    "/artificial-intelligence-1"
]
+++

사람처럼 행동하는것? 사람처럼 생각하는것? 무엇이 *AI* 일까?

> Act Rationally

여기서 *rational* 은

- Maximally achieving pre-defined goals
- Rationality only concerns what decisions are made. not the thought process begind them
- Goals are expressed in terms of the utility of outcomes

따라서 *rational* 의 의미는 **maximizing your expected utility** 

<br/>

### Brain

인간의 뇌는 *rational decision* 을 내리는데 상당히 뛰어나지만, 완벽하지는 않다. 이런 *brain* 를 모방해서 인공지능을 만들어보려 했지만 뇌는 *software* 만큼 *modular* 하지 않기 때문에 *reverse engineering* 해서 인공지능을 만들긴 어려웠다.

과학자들이 뇌를 분석하는 과정에서 얻은 *lessons learned* 는 **memory** 와 **simuation** 이 *decision making* 에서 아주 중요한 요소라는 것이다.

<br/>

### What Can AI Do?

(1) **Language**

- Translation

(2) **Vision (Perception)**

- Object and face recognition
- Scene segmentation
- Image classification

(3) **Robotics**

- Vehicles
- Rescure
- Soccer!
- Lots of automations

(4) **Logic**

- Theorem provers
- NASA fault diagnosis
- Question answering

(5) **Game Playing**

(6) **Decision Making**

- Scheduling (e.g airline routing)
- Route Planning (e.g Google maps)
- Medical diagnosis
- Web search engines
- Spam classifiers
- Automated help desks
- Fraud dections
- Productrecommendations
- Lots more!

<br/>

### Designing Rational Agents

CS188 에서는 *rational agent* 를 디자인하는 방법을 배운다. *rational agent* 란 *행동(act)* 과 *인지 (perceive)* 를 할 수 있는 *개체 (entity)* 다. 즉, *환경 (environment)* 를 인식해서 그에 맞는 판단을 내린 후 행동하는 소프트웨어로 볼 수 있다. 

- A **rational agent** selects actions that maximize its (expected) **utility**
- Characteristics of the **percepts**, **environment**, and **action space** dictate techniques for selecting rational actions

이 수업에서 배우는 것은 어떤 *environment* 와 어떤 *percept* 를  가지고 있는지를 파악한 후, 이와 관련된 기존의 테크닉을 이용해서 *언제*, *어떻게* 문제를 풀 수 있는지를 배운다.

<br/>

### Course Topics

Part 1: **Making Decisions**

- Fast search / planning
- Constraint satisfaction
- Adversarial and uncertain search

Part 2: **Reasoning under Uncertainty**

- Bayer' nets
- Decision theory
- Machine learning

Throughout: **Applications**

- Natural language processing
- Vision
- Robotics
- Games

<br/>

### Refs

(1) **Artificial Integelligence (CS 188)** by *Dan Klein, Pieter Abbeel*    
(2) [Title Image](http://www.land-of-web.com/inspiration/photography/meet-the-danbo-cute-little-cardboard-robot-photos.html)  

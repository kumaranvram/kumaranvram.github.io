---
layout: post
title: "Functional Reactive Programming - Part 1 - Introduction"
date: 2014-10-10 12:26
comments: true
categories: [functional, frp, reactive, programming]
---
Before we start with what Reactive Programming is, let us see how programming has evolved. Typically, a program is written to accomplish a certain task. To accomplish that task, the programmer writes a set of code or instructions, which when execute in that order produces the desired result. 
<!-- more -->
Here the programmer focusses on **how** the program is written. In other words, programmer tells the machine in each step how the problem should be solved. This is called as *imperative* *programming*.

On the contrary, *Declarative* programming frees the programmer from the minute details of the how and rather just tells the machine **what** to do to achieve the desired result. For example, think of SQL. The developer only tells the machine what to do as below:
```sql
SELECT blood_group, city, count(id) AS "Number of donors" 
	FROM donors 
	WHERE blood_group IN ('A+', 'B+', 'O+') 
	GROUP BY blood_group, city 
	HAVING count(id) > 0
```
Can you think of writing this logic in any imperative language like Java or C#? That would take a lot of lines of code, wouldn't it? 

That is what Declarative programming paradigm brings to the table.

### Functional Reactive Programming
Functional Reactive programming is a paradigm which combines two declarative programming paradigms *functional* and *reactive*. Let us look at each individually.

####Reactive Programming
If you have tried reading about an example for Reactive programming, this might be a cliché to you. Let us think of a spreadsheet and assume that cell C1 is given the formula **=A1+B1**. Now everytime the values of cell A1 or B1 change, the value of cell C1 automatically changes. The cell C1 reacts to the changes of A1 and B1. We do not have to reassign the formula everytime. In short, we declare the value of C as A + B and we rely on the truthfulness of the result, without paying much attention to how the value is computed.

#### Functional Programming
Functional programming is a paradigm with which we can structure our programs treating functions as our first class objects. These functions are immutable. A function is always expected to return the same output for the same input. Always. This may give jitters to our thought because we, as programmers, are used to the mutable state of objects and our programs highly rely on that.

This is where Functional Reactive Programming kicks in, giving us the best of the both worlds, imperative and functional. FRP treats user inputs as functions which are mutable, that is, whose values change over time. This requires us to step us from the comfort zone of imperative programming and requires us to change our mindset to adapt FRP to be more efficient. 

Let us discuss more on reactive programming in further chapters with greater focus on Reactive Programming in iOS. 
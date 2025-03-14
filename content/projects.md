+++
title = "Projects"
layout = "projects"
slug = "projects"
+++

## What I'm working on

{{< project 
    title="Systems Research"
    image="/img/projects/research.png"
    tags="Machine Learning,Systems,Python,C,Bash" >}}
    
I work with [Professor
Appavoo](https://www.cs.bu.edu/~jappavoo/jappavoo.github.com/index.html) and his
graduate students on his lab's research into using a program's low level binary
representation, in the form of [state
vectors](https://www.cs.bu.edu/~jappavoo/Resources/Papers/PSML/asplos14.pdf), to
learn its execution and improve performace. I have built tools to collect data
on the [Microchess](https://benlo.com/microchess/index.html) program and train
neural networks, and am developing a state vector library in C.

{{< /project >}}

{{< project 
    title="hex-calc"
    image="/img/projects/hex-calc.jpg"
    tags="Embedded,C,Electronics" >}}
Simple Hex/Binary/Decimal calculator inspired by HP-16C and TI
Programmer. Soldered and programmed a calculator designed by
[Eric Hazen](https://edf.bu.edu/ESH/). Currently implementing "woodworking mode"
to convert between metric and imperial units on an ATMega328P.
{{< /project >}}

{{< project 
    title="Autocom"
    image="/img/projects/autocom.gif"
    repo="https://marketplace.visualstudio.com/items?itemName=pbrowne011.autocom"
    tags="TypeScript,VS Code,LLM" >}}
Autocom is a VS Code extension that automatically generates both natural
language comments and Doxygen documentation for your code using AI models such
as GPT-4 and Claude Sonnet 3.5.
{{< /project >}}

{{< project 
    title="StelluxOS"
    image="/img/projects/stellux-xhci-run.png"
    repo="https://github.com/FlareCoding/StelluxOS/"
    tags="ARM64,C,C++,Operating systems" >}}
Porting StelluxOS to ARM64. StelluxOS is an operating system that implements
dynamic privilege, enabling userspace threads to elevate to kernel level and
allowing for modular kernel components that can be monitored and hot-swapped
without system crashes.
{{< /project >}}


## What I've worked on

{{< project 
    title="DFA Simulator"
    image="/img/projects/dfa.png"
    repo="https://github.com/pbrowne011/dfa"
    tags="C,Automata Theory" >}}
A program written in C that takes a defined .dfa file and an input string, and
returns whether that input string is accepted or not by the given dfa. I may
add NFA functionality to this as well at some point.
{{< /project >}}

{{< project 
    title="Cribdle"
    image="/img/projects/cribdle.png"
    repo="https://cribdle.com"
    tags="Website,JS" >}}
Cribdle is a cribbage game based on Wordle. Players aim to maximize their point
total by selecting the best cards to discard.
{{< /project >}}

{{< project 
    title="Cubert"
    image="/img/projects/cubert.png"
    repo="https://github.com/pbrowne011/cubert"
    tags="Python,Algorithms" >}}
A project to implement different algorithms to solve a Rubik's cube, each with a
different definition of "efficiency". I implement a layered algorithm approach,
and am working on building an implementation of the Thistlethwaite algorithm.
{{< /project >}}

{{< project 
    title="VaR"
    image="/img/projects/var.png"
    repo="https://github.com/pbrowne011/understandingvar"
    tags="Finance,Python" >}}
A simulation of a VaR calculation for the BU Investment club portfolio using
the historical, variance-covariance, and Monte Carlo methods. This seems a bit
more relevant given the banking issues of the first few months of 2023...
{{< /project >}}

{{< project 
    title="Turing House"
    image="/img/projects/turinghouse.gif"
    tags="Flask,Twilio,GPT-3,Java (Processing)" >}}
A hackathon project where our team built a 2D dungeon crawler game in Java
Processing. I built the backend, a Flask app that used the Twilio API to send
messages to the player as they advanced through the game, giving clues and
(hopefully) encouraging messages. I also built a chatbot interface to have a
conversation with the "ghost" of Alan Turing using the davinci-003 gpt model
from OpenAI.

Our team ended up winning the Twilio Award for our project.
{{< /project >}}

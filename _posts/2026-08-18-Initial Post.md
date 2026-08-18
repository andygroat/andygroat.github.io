---
title: "Initial Post"
date: 2026-08-18
---

# Intro

This is the first post in series of .NET development related posts. It is a series that will explore the journey of building up an API from scratch using ASP.NET Core.  There will be two repos that will hold the code for these posts, each with a different architecture, one with a classic n-tier type approach, and one with a vertical slice architecture (VSA).

# Repos

The APIs in these repos will all have the same functionality (a very simple todo item API, but will be built up to include more functionality), so if you are coming from a certain background, this will hopefully make it easy to compare the differences and similarities between the architectures.

Each repo has it's own readme file with more details of each, so the below is very quick intro to them.

## Classic N-Tier Architecture

The classic n-tier architecture splits the application into seperate logical layers. The classic n-tier type repo has layers for:

- Endpoints
- Services
- Factories

This project will start in very old school pattern, based on how I have previously developed software, and this project will go on process of modernisation, bringing in some newer techniques and styles.  The purpose of this will be to show how older APIs that have made not had much lover and kept pace with updates to .NET can be updated with some simple steps.

[Repo Link](https://github.com/andygroat/todo-classic-api)

## Vertical Slice Architecture (VSA)

The vertical slice architecture organises the code by business features, with each slice containing the all of logic required for a specific request. Its comes with the advantage over classic n-tier approach by not having to modify multiple layers to implement a feature.

[Repo Link](https://github.com/andygroat/todo-vsa-api)

# Next Steps

The next piece to look at before adding new functionality to these APIs is to add testing.  For the classic n-tier approach we will make use of NUnit and NSubstitute, which have been my traditional go-to testing and are mature libraries.  For the VSA approach, I am going to jump into a more modern testing framework and giving TUnit a go.
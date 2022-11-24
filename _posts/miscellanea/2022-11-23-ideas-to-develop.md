---
layout: post
category: Miscellanea
title: Ideas to develop
status: draft
---

- data design
  - antipatterns 
    - flat case classes structure
    - lots of enum types for typing (product and sum types are just academic mambo jambo)
    - if we can use a raw type (String, Int, ...) why bother creating a specific type
    - optional fields give you bonus points
    - the backend types should reproduce exactly the structure of the frontend types, otherwise we are duplicating efforts
- basic logging
  - log the boundaries with consistency
  - prefer blacklisting, if possible using a type class
  - implement a correlation id
- better testing
  - learn your library: custom matchers
  - unit != class
  - don't go crazy with mocks

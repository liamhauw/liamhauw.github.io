---
title: 'Home'
date: 2025-10-26
type: landing

design:
  spacing: "6rem"

sections:
  - block: hero
    content:
      title: Graphics engineering
      text: Exploring the world of fluid simulation, real-time rendering, and game engine 🚀
      primary_action:
        text: Browse docs
        url: /docs/
        icon: rocket-launch
      secondary_action:
        text: View projects
        url: /projects/
      announcement:
        text: "Welcome to my personal website"
        link:
          text: "Learn more about me"
          url: "/about/"
    design:
      spacing:
        padding: [0, 0, 0, 0]
        margin: [0, 0, 0, 0]
      css_class: ""
      background:
        color: ""
        image:
          filename: ""
          filters:
            brightness: 0.5
  - block: stats
    content:
      items:
        - statistic: "Docs"
          description: |  
            Technical knowledge base
        - statistic: "Projects"
          description: |
            Implementation showcase 
        - statistic: "Blogs"
          description: |
            Personal thoughts & stories
    design:
      css_class: "bg-gray-100 dark:bg-gray-800"
      spacing:
        padding: ["1rem", 0, "1rem", 0]
  - block: features
    id: features
    content:
      title: Interests
      text: Include fluid simulation, real-time rendering and game engine.
      items:
        - name: Fluid simulation
          icon: beaker
          description: Studying computational fluid dynamics (CFD) for realistic simulation of smoke, water, and fire. Exploring Eulerian and Lagrangian methods, Navier-Stokes solvers, and advanced techniques like FLIP, PIC, and SPH for visually compelling fluid behaviors.
        - name: Real-time rendering
          icon: bolt
          description: Researching real-time rendering techniques including physically-based rendering (PBR), global illumination, ray tracing, and modern GPU-accelerated algorithms. Focus on achieving photorealistic visuals while maintaining interactive frame rates.
        - name: Game engine
          icon: cube
          description: Exploring modern game engine design patterns including Entity-Component-System (ECS), multithreaded rendering pipelines, asset management, and data-driven workflows. Building scalable and performant systems for interactive 3D applications.
  - block: cta-card
    content:
      title: "Start exploring the fascinating world of graphics engineering"
      button:
        text: Reading docs
        url: /docs/
    design:
      card:
        css_class: "bg-primary-700"
        css_style: ""
---

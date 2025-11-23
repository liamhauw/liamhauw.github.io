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
      text: Exploring the world of physical simulation, real-time rendering, game engine, and deep learning 🚀
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
      text: Include physical simulation, real-time rendering, game engine, and deep learning.
      items:
        - name: Physical simulation
          icon: beaker
          description: Mastering computational fluid dynamics (CFD) and rigid body dynamics to create lifelike simulations of smoke, water, fire, and solid objects. Leveraging Eulerian and Lagrangian approaches, Navier-Stokes solvers, and cutting-edge techniques including FLIP, PIC, SPH for fluid behaviors, alongside sophisticated collision detection and response systems for rigid body interactions.
        - name: Real-time rendering
          icon: bolt
          description: Advancing real-time rendering through physically-based rendering (PBR), global illumination, ray tracing, and GPU-accelerated algorithms. Dedicated to achieving photorealistic visuals at interactive frame rates by optimizing rendering pipelines and embracing next-generation graphics technologies for immersive visual experiences.
        - name: Game engine
          icon: cube
          description: Architecting robust game engine systems with modern design patterns such as Entity-Component-System (ECS), multithreaded rendering pipelines, efficient asset management, and data-driven workflows. Focused on building scalable, high-performance frameworks that power interactive 3D applications and deliver seamless real-time experiences.
        - name: Deep learning
          icon: cpu-chip
          description: Applying deep learning and neural network architectures to revolutionize computer graphics and vision tasks. Exploring innovative applications in image synthesis, style transfer, neural rendering, and AI-driven content creation to enhance visual fidelity, accelerate workflows, and unlock new creative possibilities through intelligent automation.
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

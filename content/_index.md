---
title: 'Home'
date: 2025-12-21
type: landing

design:
  # Default section spacing
  spacing: "6rem"

sections:
  - block: hero
    content:
      title: Less is more
      text: A personal hub for technical docs, engineering projects, and daily blogs.
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
      # For full-screen, add `min-h-screen` below
      css_class: ""
      background:
        color: ""
        image:
          # Add your image background to `assets/media/`.
          filename: ""
          filters:
            brightness: 0.5
  - block: stats
    content:
      items:
        - statistic: "Docs"
          description: |
            Technical notes and reference
        - statistic: "Projects"
          description: |
            Engineering work and prototypes
        - statistic: "Blogs"
          description: |
            Reflections and perspectives
    design:
      # Section background color (CSS class)
      css_class: "bg-gray-100 dark:bg-gray-800"
  - block: features
    id: features
    content:
      title: Focus Areas
      text: Core domains across graphics engineering.
      items:
        - name: Physical simulation
          icon: beaker
          description: Physically-based simulation workflows for interactive applications, with solver efficiency and numerical robustness in mind.
        - name: Real-time rendering
          icon: bolt
          description: Real-time pipelines focused on image quality, frame stability, and efficient GPU execution.
        - name: GPU
          icon: server
          description: GPU architecture, shader/compute execution models, and performance analysis for real-time workloads.
        - name: Vulkan
          icon: queue-list
          description: Explicit graphics API engineering such as resource management, synchronization, and frame graph style render orchestration.
        - name: Game engine
          icon: cube
          description: Runtime architecture and tooling for scalable, data-driven, and performance-critical interactive systems.
        - name: Deep learning
          icon: cpu-chip
          description: Applying neural networks to vision and graphics-adjacent tasks with practical, production-minded pipelines.
  - block: cta-card
    content:
      title: "One more thing"
      text: "“The end of labor is to gain leisure.” — Aristotle"
      button:
        text: Read the blogs
        url: /blogs/
    design:
      card:
        # Card background color (CSS class)
        css_class: "bg-primary-700"
        css_style: ""
---

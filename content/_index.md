---
# Leave the homepage title empty to use the site title from params.yaml
title: ''
summary: ''
date: 2026-01-05
type: landing

sections:
  # ──────────────────────────────────────────────────────────────────────────────
  # HERO  —  gradient background, name, role, social links, typewriter, CTAs
  # ──────────────────────────────────────────────────────────────────────────────
  - block: dev-hero
    id: hero
    content:
      username: me
      greeting: "Hi, I'm"
      show_status: true
      show_scroll_indicator: true
      typewriter:
        enable: true
        prefix: "I build"
        strings:
          # TODO: replace with phrases that match your work
          - "interpretable ML for healthcare"
          - "scalable bioinformatics pipelines"
          - "single-cell analysis workflows"
          - "clinical genomics tools"
        type_speed: 70
        delete_speed: 40
        pause_time: 2500
      cta_buttons:
        - text: View My Work
          url: "#projects"
          icon: arrow-down
        - text: Get In Touch
          url: "#contact"
          icon: envelope
    design:
      style: centered
      avatar_shape: circle
      animations: true
      background:
        color:
          light: "#fafafa"
          dark: "#0a0a0f"
      spacing:
        padding: ["6rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # ABOUT  —  about section
  # ──────────────────────────────────────────────────────────────────────────────
  - block: resume-biography
    id: about 
    content:
      username: About Me  # Points to content/authors/admin/
      title: About Me
      text: |
        Innovative Computational / Molecular Biologist with 5+ years of experience generating and handling Biological Big Data. I have designed pipelines for streamlined analysis of NGS data assisted by bioinformatics and machine learning algorithms that have analyzed Terabytes of data.

        Technical Skills: Python, R, Bash, JavaScript, SQL.

        Here are a few examples of my work:

        •	Employed deep neural learning algorithms to predict mutations in SARS-CoV-2 and accurately forecasted the change in the viral genome by 17.4% over 2000 variants.

        •	Developed a Python program to process, clean, and perform analytics on raw data from cellular measurements with digital imaging, reducing time constraints from 2 days to 2 hours.

        •	Designed potential vaccines and biomarkers based on computational protein and cellular modeling that had over 75% population coverage which led to over 5 research articles in Q1 Journals.

        Making sense of biological big data using my interdisciplinary skills for real human impact gets my gears going.

        Always happy to connect with new people.
    design:
      id: about        # Anchor link ID for your navigation menu

  # ──────────────────────────────────────────────────────────────────────────────
  # PROJECTS  —  filterable portfolio grid (Alpine.js powered)
  # One markdown file per project in content/projects/
  # ──────────────────────────────────────────────────────────────────────────────
  - block: portfolio
    id: projects
    content:
      title: "Featured Projects"
      subtitle: "A selection of my recent work"
      count: 0                                  # 0 = show all
      filters:
        folders:
          - projects
      buttons:
        - name: All
          tag: '*'
        - name: Single-cell Genomics
          tag: Single-cell Genomics
        - name: Epigenomics
          tag: Epigenomics
        - name: Single-cell Multiomics
          tag: Single-cell Multiomics
        - name: Spatial Transcriptomics
          tag: Spatial Transcriptomics
        - name: Structural Proteomics
          tag: Structural Proteomics
        - name: Machine Learning
          tag: Machine Learning
      default_button_index: 0
    design:
      columns: 3
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # TECH STACK  —  icons grouped by category
  # Icon names use the devicon/ or brands/ namespace (see devicon.dev)
  # ──────────────────────────────────────────────────────────────────────────────
  - block: tech-stack
    id: skills
    content:
      title: "Tech Stack"
      subtitle: "Technologies I use to build things"
      categories:
        # TODO: edit to match your real toolkit
        - name: Languages
          items:
            - name: Python
              icon: devicon/python
            - name: R
              icon: devicon/r
            - name: Bash
              icon: devicon/linux
            - name: SQL
              icon: devicon/postgresql
        - name: Bioinformatics
          items:
            - name: Nextflow
              icon: devicon/bash
            - name: Snakemake
              icon: devicon/python
            - name: Bioconductor
              icon: devicon/r
            - name: samtools
              icon: devicon/linux
        - name: ML & Data
          items:
            - name: scikit-learn
              icon: devicon/python
            - name: PyTorch
              icon: devicon/pytorch
            - name: pandas
              icon: devicon/python
            - name: NumPy
              icon: devicon/python
        - name: Cloud & DevOps
          items:
            - name: Docker
              icon: devicon/docker
            - name: AWS
              icon: devicon/amazonwebservices
            - name: GitHub Actions
              icon: brands/github
            - name: Slurm
              icon: devicon/linux
        - name: Visualization
          items:
            - name: ggplot2
              icon: devicon/r
            - name: matplotlib
              icon: devicon/python
            - name: seaborn
              icon: devicon/python
            - name: Plotly
              icon: devicon/python
    design:
      style: grid
      show_levels: false
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # EXPERIENCE  —  chronological timeline
  # TODO: replace each item with your real roles
  # ──────────────────────────────────────────────────────────────────────────────
  - block: resume-experience
    id: experience
    content:
      title: Experience
      date_format: Nov 2025
      items:
        - title: Bioinformatics Research Scientist
          company: St. Jude Children's Research Hospital
          company_url: 'https://www.stjude.org/'
          company_logo: ''
          location: Memphis, TN
          date_start: '2025-11-01'
          date_end: ''
          description: |2-
            * Develop and maintain reproducible genomics analysis pipelines
            * Apply interpretable machine learning to clinical and omics datasets
            * Collaborate with wet-lab and clinical teams on study design
            * Tech stack: Python, R, Nextflow, Docker, AWS
        - title: Research Associate
          company: TODO — previous employer
          company_url: ''
          company_logo: ''
          location: TODO
          date_start: '2018-01-01'
          date_end: '2020-12-31'
          description: |2-
            * TODO — describe your contributions and impact
            * TODO — quantify results where possible
            * Tech stack: TODO
    design:
      columns: '1'
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # ACHIEVEMENTS  —  chronological timeline of awards
  # ──────────────────────────────────────────────────────────────────────────────
  - block: resume-awards
    id: achievements
    content:
      title: Achievements
      subtitle: Professional milestones and credentials
      date_format: Jan 2006
      items:
        - title: AWS Certified Solutions Architect
          organization: Amazon Web Services
          organization_url: https://amazon.com


          date_start: '2026-01-15'
          description: Mastered cloud architecture, security, and deployment strategy.
          certificate_url: https://example.com
          icon: aws
          
        - title: Winner - Global Hackathon 2025
          organization: TechCorp
          organization_url: https://example.com
          date_start: '2025-11-01'
          description: Built an AI-driven dev tool that took 1st place out of 500 teams.
          icon: trophy
  
  # ──────────────────────────────────────────────────────────────────────────────
  # STATS  —  about section
  # ──────────────────────────────────────────────────────────────────────────────
  # - block: stats
  #   content:
  #     title: Impact Metrics
  #     text: Quantifiable milestones across my engineering career.
  #     items:
  #       - statistic: '50k+'
  #         description: Active App Downloads
  #         icon: hero/arrow-down-tray
  #       - statistic: '3'
  #         description: Open Source Tools Built
  #         icon: hero/code-bracket
  #       - statistic: '100%'
  #         description: Test Coverage Maintained
  #         icon: hero/check-circle
  #   design:
  #     layout: cards

  # ──────────────────────────────────────────────────────────────────────────────
  # BLOG  —  recent posts (one markdown file per post in content/blog/)
  # ──────────────────────────────────────────────────────────────────────────────
  - block: collection
    id: blog
    content:
      title: Recent Posts
      subtitle: 'Thoughts on bioinformatics, ML, and reproducible research'
      text: ''
      filters:
        folders:
          - blog
        exclude_featured: false
      count: 3
      order: desc
    design:
      view: card
      columns: 3
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # CONTACT  —  email + social links
  # ──────────────────────────────────────────────────────────────────────────────
  - block: contact-info
    id: contact
    content:
      title: Get In Touch
      subtitle: "Let's build something amazing together"
      text: |-
        I'm open to industry data-science and bioinformatics projects.
        Whether you're hiring, collaborating, or just want to talk shop, feel free to reach out.
      email: hasan.al.reza.bd@gmail.com              # TODO: your real email
      autolink: true
    design:
      columns: '1'
      background:
        color:
          light: "#ffffff"
          dark: "#0d0d12"
      spacing:
        padding: ["4rem", "0", "4rem", "0"]

  # ──────────────────────────────────────────────────────────────────────────────
  # CTA CARD  —  "Open to Opportunities" + resume download
  # ──────────────────────────────────────────────────────────────────────────────
  - block: cta-card
    content:
      title: "Interests"
      text: |-
        I'm interested in **bioinformatics** or **data science** projects.

        Let's connect and discuss how I can help your team.
      button:
        text: 'Download Resume'
        url: uploads/CV_HAR_F7MD.pdf                 # TODO: place your PDF at static/uploads/resume.pdf
        new_tab: true
    design:
      card:
        css_class: 'bg-gradient-to-br from-primary-200 via-primary-100 to-secondary-200 dark:from-primary-600 dark:via-primary-700 dark:to-secondary-700'
        text_color: dark
      background:
        color:
          light: "#f5f5f5"
          dark: "#08080c"
      spacing:
        padding: ["4rem", "0", "6rem", "0"]
---

---
title: "Me"
layout: "me"
url: "/me/"
summary: "About Muhammad Ramzan"

tagline: "BS Software Engineering student, competitive programmer, backend engineer and embedded systems enthusiast"

contact:
  - label: "mr.ramzan.bhutto@gmail.com"
    url: "mailto:mr.ramzan.bhutto@gmail.com"
    icon: "fa-solid fa-envelope"
  - label: "LinkedIn"
    url: "https://www.linkedin.com/in/ramzan-bhutto/"
    icon: "fa-brands fa-linkedin"
  - label: "GitHub"
    url: "https://github.com/ramzanbhutto"
    icon: "fa-brands fa-github"

skills:
  - category: "Languages"
    items: ["C", "C++", "Go", "C#", "Python", "Assembly", "HTML", "CSS", "JavaScript"]
  - category: "Frameworks"
    items: ["ASP.NET Core", "React", "Blazor Web App", "Avalonia UI"]
  - category: "Databases"
    items: ["MySQL", "PostgreSQL", "MongoDB"]
  - category: "Cloud"
    items: ["AWS EC2", "AWS S3", "AWS Lambda"]
  - category: "DevOps & Tools"
    items: ["Git", "GitHub", "GitLab", "n8n", "Docker", "Kubernetes", "GTest", "xUnit", "GitHub Actions", "GitLab CI/CD", "KVM", "QEMU", "virt-manager", "CMake", "Linux", "Zephyr RTOS"]
  - category: "Other"
    items: ["Hugo", "Doxygen", "LaTeX", "Neovim", "Visual Studio"]

education:
  - school: "FAST NUCES Peshawar"
    degree: "BS Software Engineering"
    date: "Expected May 2028"
    detail: "CGPA: 3.16 / 4.0"
  - school: "Cadet College Sanghar"
    degree: "Pre-Engineering"
    date: "May 2024"
    detail: "87.7%"

projects:
  - name: "crun"
    stack: "C++, Linux, CMake"
    link: "https://github.com/ramzanbhutto/crun"
    points:
      - "Building a Linux container runtime like Docker from scratch, using namespaces and pivot_root to isolate a container's process tree, filesystem and hostname from the host."
      - "Wrote a cgroup v2 RAII wrapper to cap CPU and memory per container, with cleanup guaranteed even if the process exits early."
      - "Designed a socketpair-based handshake between host and container init process, so the host finishes setup before the container's main process starts, avoiding race conditions."
  - name: "Rendered Markdown Blog Editor"
    stack: "Node.js, Express, JavaScript"
    link: "https://github.com/ramzanbhutto/Rendered-Markdown"
    points:
      - "Built an offline blog editor with a Node.js and Express backend that persists posts as Markdown files on disk instead of a database, with a REST API for create, read, update, delete and import."
      - "Designed a custom file format that embeds JSON metadata above a separator in each Markdown file, and added live preview with a custom marked.js renderer that rewrites image paths for offline asset embedding."
  - name: "LibVLCSharp DVD Player"
    stack: "C#, Avalonia UI, LibVLCSharp"
    link: "https://github.com/mfkl/libvlcsharp-samples/pull/223"
    points:
      - "Built a cross-platform DVD player desktop app in C# with full menu navigation, verified end-to-end on Linux and macOS."
      - "Contributed it to the official libvlcsharp-samples repo as part of GSoC 2026 pre-selection work."
  - name: "Ray Tracer"
    stack: "C++, CMake"
    link: "https://github.com/ramzanbhutto/Ray-Tracer"
    points:
      - "Wrote a ray-sphere renderer from scratch that renders an 800x450 scene in ~0.3s using a simplified quadratic intersection test."
      - "Modeled the scene as Hittable objects with polymorphism, tracking hit point, surface normal, and ray direction."
  - name: "Alumni Management System"
    stack: "C#, .NET 10, Blazor, EF Core, MySQL"
    link: "https://github.com/ramzanbhutto/AlumniManagementSystem"
    points:
      - "Built a full-stack alumni portal in Clean Architecture with CQRS, with 12 REST API controllers across 16 domain entities and role-based access for Admin, Alumni, and Guest."
      - "Secured auth with JWT (HMAC-SHA256, 60-minute expiry) and BCrypt hashing at work factor 12, plus three rate limiting policies to block brute-force attempts on login and heavy report endpoints."
  - name: "DomJudge Contest Platform"
    stack: "Docker, Docker Compose, Bash"
    link: "https://ramzanbhutto.github.io/posts/domjudge1/"
    points:
      - "Set up and hosted ICPC-style contests on DomJudge, containerizing MariaDB, DOMserver, and multiple judgehost instances, with a bash script that auto-detects CPU cores to spin up one judgehost per core and generate the Docker Compose config."
      - "Prepared problems on Polygon with testlib.h validators, packaged them in the ICPC Problem Package Format and exposed the contest to remote participants over a Cloudflare Tunnel."

achievements:
  - title: "ICPC Asia Topi Regional"
    detail: "Qualified and competed onsite"
    date: "2025"
  - title: "Codeforces Pupil"
    detail: "First from FAST NUCES Peshawar to reach Pupil — Rating 1368, handle: mr_ramzan"
    date: "2026"
  - title: "1st Place — Competitive Programming Competition"
    detail: "FAST NUCES"
    date: "2025"
  - title: "Merged PR at Ventoy"
    detail: "Added full Urdu language support to the bootable USB tool"
    date: "2025"
  - title: "2nd Place — All Pakistan Essay Writing Competition"
    detail: "APEITC"
    date: "2023"
---

---
title: 'Nebula Flow Walkthrough: How I Built a Realtime 3D Hand-Reactive Universe Using Just Prompts'
description: >-
  A raw breakdown of how I used Google AI Studio to generate a full 3D particle
  experience: Nebula Flow: with hand tracking, AI-generated presets, and 
  shader-driven visuals, without manually coding the whole thing.
categories:
  - AI
  - Projects
  - WebGL
tags:
  - AI Studio
  - Three.js
  - R3F
  - MediaPipe
  - Gemini
date: '2025-12-03'
published: true
layout: post
---

> project: **Nebula Flow** <br>
> honestly… AI today is doing things we aren't even fully skilled for yet: the only missing skill is knowing *how* to talk to it.  <br>
> this whole thing… built by prompting. not hand-coding. wild times :><br>
> demo at: [https://particals.lavsarkari.me/](https://particals.lavsarkari.me/) <br>
> codes at: [https://github.com/LavSarkari/PARTICELS-](https://github.com/LavSarkari/PARTICELS-) <br>

---

## before anything: what i actually wanted

i had a picture in my head: a **floating 3D nebula** moving like soft cosmic smoke…  <br>
and when you move your hand in front of the camera:<br>

* open palm → the nebula **pushes away**  <br>
* closed fist → the nebula **collapses into a black hole**  <br>
* two hands → you can **stretch** the galaxy apart  <br>

and then a small input box where i type:<br>

> “matrix rain”  <br>
> “cotton-candy nebula”  <br>
> “golden fireflies vortex”<br>

and **Gemini** redesigns the entire particle system on the fly.<br>

i didn’t want to write a giant three.js shader pipeline manually.<br>

i wanted a **prompt** that makes AI build everything start-to-finish.<br>

---

## step 1: accepting the truth: AI is literally our senior dev now<br>

i stopped pretending i’ll hand-craft a thousand-line project.<br>

i opened Google AI Studio, sat back, and treated it like hiring a pro:<br>

> “build my whole app, file by file, ready for Vercel.”<br>
once you start prompting like you're giving requirements to a real dev,  <br>
AI stops being a toy and starts becoming a **full-stack teammate**.<br>

---

## step 2: the master prompt: the moment everything was born

i wrote *one* big instruction block that basically shaped the entire universe.<br>

this is the exact thing i fed into AI Studio:<br>

```
Create a complete, production-ready React application called "Nebula Flow" using Vite, TypeScript, Three.js (React Three Fiber), and Tailwind CSS.
Core Functionality:
3D Particle System: Build a high-performance ParticleField component using THREE.Points and custom GLSL shaders. The particles should feature:
Perlin noise-based movement.
Dynamic interactions: Repel on open hand/mouse, Attract (black hole effect) on fist.
Visual polish: Particles should fade out at the edges, glow brighter based on their speed/energy, and mix between two colors.
AI Generation: Integrate the Google Gemini API (@google/genai) to generate particle configurations (count, speed, colors, noise strength) based on natural language user prompts (e.g., "Matrix Rain", "Golden Fireflies").
Real-time Hand Tracking: Use MediaPipe (@mediapipe/tasks-vision) to implement a HandTracker component that:
Tracks up to 2 hands via webcam.
Detects gestures: Fist (Attract), Open Palm (Repel), and Pinch.
Dual-hand logic: Calculate the distance between two hands to control the "Dispersion/Zoom" of the nebula in real-time.
Includes a futuristic debug overlay showing the video feed with drawn hand skeletons.
Architecture:
App.tsx: Manages state between the UI, the 3D scene, and the hand tracker. Use useRef to pass hand coordinates to the animation loop without re-renders.
Controls.tsx: A glassmorphism UI panel to toggle inputs (Mouse vs Camera), adjust sliders manually, and input AI prompts.
Services: A helper to call Gemini with a structured JSON schema.
Requirements:
Handle process.env safely for Vercel deployment.
Use lucide-react for icons.
Ensure the GLSL shader compiles correctly for WebGL 2.0.
Include a metadata.json requesting camera permissions.
```


the crazy part?  <br>
**AI understood every part of that like a senior engineer.**<br>

it created:<br>

* folders  <br>
* files  <br>
* structure  <br>
* shaders  <br>
* architecture decisions  <br>
* state management  <br>
* UI controls  <br>
* permission files  <br>
* deployment-safe code  <br>

all because the prompt was crystal.<br>

that's the gap most people feel:  <br>
they know AI is powerful but don’t know what words make it *unleash*.<br>

---

## step 3: the AI magic moment: watching it assemble a universe

AI Studio didn’t behave like a “text generator”.  <br>
it behaved like an engineer who understood:<br>

* how React Three Fiber works  <br>
* how to inject hand position into the animation loop  <br>
* how MediaPipe detects gestures  <br>
* how to design a glass UI panel  <br>
* how to structure a Vite + TS project  <br>
* how to make Gemini return structured JSON  <br>
* how to keep it WebGL2-compatible  
<br>
the more precise I went with the prompt, the more it exploded into solid output.<br>

that’s when it hit me:<br>

> we don’t lack skills.  <br>
> we lack vocabulary to command these models.  <br>
> tech is no longer about “knowing everything”:  <br>
> it’s about describing what you want so precisely that AI executes it like code.<br>

---

## step 4: iterative refinement: the real secret sauce

AI didn’t get everything perfect first try.  
but every time something broke, I simply pasted the error:

> “hand tracker not reading both hands: fix the gesture logic”  
> “shader failing in WebGL2: rewrite compatible version”  
> “particles popping out of bounds: stabilise dispersion force”  

and the model handled it like debugging with a teammate who never gets tired.

this stage is where most people give up because they think:

> “AI made a mistake → AI is dumb”

nah.  
AI is the most obedient engineer you’ll ever work with.  
you just need to know how to *correctly scold it*.

---

## step 5: deployment: it literally built itself for Vercel

i asked:

> “prepare the project so I can deploy on Vercel without config issues”

it:

* created `.env.example`  
* made Gemini keys safe  
* added proper vite configs  
* fixed camera permission metadata  
* ensured build-safe imports  
* removed browser-unsafe APIs  

after that…
```terminal
npm run build
```

vercel deploy

nebula online ✨

---

## why this matters (real talk)

**Nebula Flow** wasn’t about particles for me.  
it was me testing a theory:

> can you build complex, interactive, AI-enhanced, real-time graphics apps
> **purely through prompting?**

answer:  
**yes. absolutely yes.**

AI today already knows:

* graphics pipelines  
* GPU shaders  
* R3F structure  
* hand tracking architecture  
* UI design patterns  
* state management logic  
* async APIs  
* env-handling for deployment  

the only thing it doesn’t know is:

> what *you* want.

if you learn how to express requirements like an engineer,  
AI does the engineering for you.

Nebula Flow became proof of that.

---

## closing thought: the real skill in 2025

it’s not:

* learning every framework  
* mastering every API  
* writing boilerplate by hand  

it’s:

> **learning to prompt like a technical architect.**  
> not “generate code”…  
> but “describe systems.”

your clarity becomes its intelligence.

the future dev workflow is literally:

1. imagine something outrageous  
2. write a prompt that explains it cleanly  
3. let AI build the scaffolding  
4. refine with errors and constraints  
5. deploy

Nebula Flow was my test.  
and it worked.

if you ever thought “i can’t build this”,  
believe me: you absolutely can.

you just have to talk to AI the way engineers talk to junior devs.

the code?  
AI will handle that.

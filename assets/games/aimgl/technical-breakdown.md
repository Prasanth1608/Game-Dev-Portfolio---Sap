AimGL — Technical System Breakdown
Engine: Custom, non-ECS, OpenGL 3.3 + GLFW + Assimp + BASS audio + ImGui + GLM. ~3,700 lines of hand-written code (rest is third-party: ImGui).

1. Core Loop (Main/main.cpp)
Single while(!glfwWindowShouldClose) loop, no fixed-timestep — everything runs at render framerate. Order per frame: clear → menu-state input → FPS calc → Gameplay() (branches on state) → poll events → swap buffers. All game objects live in one global Classes struct (Window, Shaders, Textures, Rigidbody, Game, Box) — no scene graph, no entity system. Game is a god object that owns camera, targets, lighting, matrices, bullets, stats, performance, and audio as nested structs.

2. State Machine
Flat 3-state enum: menu_state_home, menu_state_game_bullet, menu_state_game_projectile. No stack, no transitions table — just if/else checks scattered through main.cpp and Game.h. RestartGame() resets score/timer/position on mode entry.

3. Rendering (Graphics/Rendering, Graphics/Objects)
Shaders.h: raw GLSL strings embedded in C++ (not external files), one shader program per draw category — bullet, box (debug wireframe), assimp (3D models), UI (2D quads). Model/Mesh: Assimp-based OBJ loader → flattens into Mesh objects with position/normal/UV + diffuse/specular/normal/height texture slots. Standard LearnOpenGL-style mesh pipeline. UI.h: hand-built 2D quads (VAO/VBO/EBO per element — home screen, gun sprite, crosshair) rendered with orthographic-style offset uniforms, no batching. Score/timer/FPS/game-mode text is a separate pass via ImGui overlaid on top of the OpenGL scene each frame.

4. Physics (Physics/Rigidbody)
Minimal, single rigidbody used for whichever bullet is currently active — not per-object. Semi-implicit Euler integration: position += velocity*dt + 0.5*accel*dt², velocity += accel*dt. Supports Force, Impulse, Acceleration, and a TransferEnergy (impulse-from-energy) method. Gravity is just vec3(0, g, 0) added as acceleration — used for Projectile mode (arced shots), zeroed out for Bullet mode (hitscan-like straight raycast). Note: only one Rigidbody instance drives bullet motion globally, and it's reset (ResetPhysics_Projectile) between shots — so it can't handle multiple simultaneous in-flight projectiles correctly (a known scaling limitation).

5. Collision (Collision/Box, Collision/CollisionRegions)
CollisionRegion: dual-mode primitive supporting AABB (min/max) and Sphere (center/radius), with ContainsPoint, ContainsRegion, RegionIntersects — full box-box, sphere-sphere, and mixed box-sphere test matrix. Textbook broad-phase collision math. Box: GPU-instanced debug-line renderer for bounding volumes (static + moving), using glVertexAttribDivisor for instancing, capped at UPPER_BOUND=100 instances. Actual hit detection (bullet-vs-target) isn't done through this system — it's a manual AABB-ish distance check hardcoded in main.cpp::DetectCollision_Bullet() (±10 unit box around target), bypassing the CollisionRegion class entirely. So there are effectively two parallel collision systems: a generic one (unused for gameplay) and an ad-hoc one (actually used).

6. Gameplay Systems
Bullet mode: instant straight-line "raycast" (no gravity, high impulse magnitude 20000). Projectile mode: arced shot with gravity (-10) and lower magnitude (7000), rendered as an animated 3D mesh following the rigidbody position. Bullets_Array: a vector of bool-flag structs tracking spawn state — used more as a firing-state tracker than true multi-bullet pooling (only bullet [0]/latest really matters given the single shared Rigidbody). Target spawner: random X/Y within a bounded rectangle, re-rolled on hit inside InitUniforms() (rendering code doing gameplay logic — tight coupling). Map boundary clamps (hardcoded world-space X/Z limits) restrict player movement.

7. Camera / Input
FPS-style mouse-look camera computed manually via yaw/pitch → direction vector (no quaternions), clamped pitch ±89°. WASD movement projects along cam direction/right vectors scaled by deltaTime.

8. Audio (Audio/Sound)
Thin wrapper around the BASS audio library — 4 fire-and-forget SFX channels (hit, shoot, game-over, bgm placeholder). No mixing/pooling beyond BASS's own channel management.

9. Third-Party Deps
GLFW (window/input), GLAD (GL loader, implied), GLM (math), Assimp (model import), stb_image (textures), BASS (audio), Dear ImGui (debug/HUD text overlay).

Overall architecture pattern: tutorial-driven, single-translation-unit-style architecture (very LearnOpenGL-influenced) — good for a student demake, but has classic anti-patterns: god object (Game), logic-in-render-function coupling, a single shared physics body instead of per-entity, hardcoded magic numbers for world bounds/hit radii, and a parallel unused collision system.

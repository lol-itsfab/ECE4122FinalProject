# ECE4122FinalProject
This project is a real-time 3D particle simulator.
It contains three modes:
1. A fountain mode
2. An explosion mode
3. A vortex mode
These modes each renders thousands of particles inside a bounding box with an interactive camera, that can be used to move around and spectate through different views.

## Build Instructions (ON Pace-ICE)
1. create a build directory after making one (i.e mkdir build and cd build)
2. Run a clean build (i.e cmake ..)
3. Compile and build (i.e cmake --build . -j8)
4. cd into the newly constructed bin folder (i.e cd bin)
5. FInally run the executable using ./Executable_name (i.e ./FinalProject)

## Controls
 
### Camera
 
| Input | Action |
|---|---|
| Left mouse drag | Orbit camera (yaw and pitch) |
| Right mouse drag | Pan the camera target |
| Scroll wheel | Zoom in / out |
| `R` | Reset camera to default position |

### Simulation
| Input | Action |
|---|---|
| `1` | Switch to **Fountain** mode |
| `2` | Switch to **Explosion** mode |
| `3` | Switch to **Vortex** mode |
| `TAB` | Cycle through emitter modes |
| `SPACE` | Pause / resume simulation |
| `ESC` | Quit |


## Implemented Features
### Class Architecture
- **`Particle`** — Represents a single particle with position, velocity, acceleration, color gradient, size, mass, age, and lifetime. Follows the rule-of-five with all special members explicitly defined or defaulted.
- **`ParticleEmitter`** — Manages a fixed-capacity particle pool using a free-list stack to recycle dead particles with zero per-frame heap allocation. Spawns particles according to the active emitter mode each frame.
- **`Renderer`** — Encapsulates all OpenGL state (VAOs, VBOs, shader programs) with full RAII ownership. The only file in the project that calls any `gl*` function.
- **`Camera`** — Arc-ball orbital camera driven entirely by mouse and keyboard input with yaw, pitch, zoom, and pan support.
- **`SimulationConfig`** — INI file parser that populates a plain-data struct at startup. Falls back to safe defaults when the file is absent or a value is malformed.
### Physics
- **Gravity** — Constant gravitational acceleration vector applied to all particles every frame (default: `{0, -9.81, 0}`).
- **Semi-implicit Euler integration** — Velocity is updated before position each frame, providing better energy conservation than classical Euler integration.
- **Bounding box collision** — Particles reflect off all six faces of a configurable axis-aligned bounding box with a tunable coefficient of restitution.
- **Floor friction** — Horizontal velocity is damped by 15% on each floor bounce to simulate energy loss on impact.
- **Particle aging and recycling** — Each particle has a randomised lifetime. Dead particles are returned to the free-list and reused the next frame without any heap allocation.
### Emitter Modes
- **Fountain** — Particles launch upward inside a 25° half-angle cone, arcing under gravity and bouncing off the floor like a water fountain.
- **Explosion** — A burst of particles (8% of the pool) fires uniformly in all directions every 3.5 seconds using correct uniform sphere sampling (`acos` method).
- **Vortex** — Particles spiral upward with randomised tangential, radial, and vertical velocity components, producing a tornado-like effect.
### Rendering
- **OpenGL 3.3 Core Profile** — No deprecated fixed-function pipeline calls.
- **Two GLSL shader programs** — Particle point-sprite shader and bounding-box wireframe shader, both loaded from `.vert` / `.frag` files at runtime.
- **Per-particle color gradient** — Particles interpolate from a warm yellow-white (birth) to a cool transparent blue (death) via `glm::mix` in the vertex shader, driven by normalised age.
- **Additive blending** — Overlapping particles brighten each other (`GL_SRC_ALPHA, GL_ONE`), producing a glowing ember effect.
- **Perspective-correct point sprites** — `gl_PointSize` is set in the vertex shader based on distance from the camera so particles appear consistently sized at any zoom level.
- **VBO orphaning** — The particle VBO is re-orphaned every frame (`glBufferData` with `nullptr`) before uploading new data, eliminating GPU pipeline stalls.
- **Bounding box wireframe** — 12-edge AABB rendered as `GL_LINES` to give the scene a clear spatial reference.
- **FPS counter** — Live frame rate, particle count, and current emitter mode displayed in the window title bar.
### Parallelism
- **OpenMP physics loop** — The per-particle physics update (`Particle::update` and `Particle::collideBounds`) runs inside a `#pragma omp parallel for schedule(static)` region. Thread count is configurable via `num_threads` in the config file.

## Screenshots(of modes)
1. 

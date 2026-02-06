# **CARET _ ROS2 Runtime Analysis using CARET (Autoware)**

## Overview

To truly master ROS2 and deploy it in real-world applications, you need more than just algorithm knowledge — you need solid engineering techniques. ROS2 provides `ros2_tracing` as a built-in library for runtime analysis.

However, `ros2_tracing` has limitations:

- No built-in visualization tools
- Difficult to use with large-scale systems
- Manual analysis required for complex node graphs

## Why CARET?

I primarily work with **Tier4's Autoware**, which consists of:

- **500+ packages**
- **100+ topics**

Debugging and analyzing such a massive system is nearly impossible with basic tools. That's where **CARET** comes in.

CARET (Chain-Aware ROS Evaluation Tool) is developed by Tier4 to provide:

- End-to-end latency analysis
- Node/topic communication visualization
- Callback execution time profiling
- Bottleneck detection in complex ROS2 graphs

## Installation

### The Problem: CARET + Autoware Environment Conflict

CARET and Autoware have different environment requirements, which causes conflicts during build. Here's what you need to know:

1. **CARET can be built without full environment setup** - Building works fine initially
2. **DO NOT source CARET before building Autoware** - Complete Autoware build first without CARET sourced
3. **If you ignore this, Autoware build will break** - CARET dependencies will mess up the entire build chain
4. **Use `local_setup.bash` when running** - This is crucial for proper environment isolation

### Why `local_setup.bash` instead of `setup.bash`?

| Script | Behavior |
|--------|----------|
| `setup.bash` | Sources current workspace + ALL underlying workspaces (chained) |
| `local_setup.bash` | Sources ONLY the current workspace |

Using `local_setup.bash` prevents environment contamination between CARET and Autoware.

### Recommended Setup

```bash
# 1. Use Python venv for CARET (Recommended)
python3 -m venv ~/caret_venv
source ~/caret_venv/bin/activate

# 2. Build Autoware FIRST (without CARET sourced)
cd ~/autoware
colcon build

# 3. Build CARET separately
cd ~/caret_ws
colcon build

# 4. When using CARET, source in this order:
source /opt/ros/humble/setup.bash
source ~/autoware/install/local_setup.bash
source ~/caret_ws/install/local_setup.bash
```

### Troubleshooting

If Autoware build gets corrupted after CARET setup, check these in order:

1. **ROS2 environment** - `echo $ROS_DISTRO`, `which ros2`
2. **Python environment** - `which python3`, check `PYTHONPATH`
3. **CUDA paths** - `echo $LD_LIBRARY_PATH`, `nvcc --version`

Rebuilding from scratch is often faster than debugging tangled environments.

### Future Plans

I'm considering submitting a **Pull Request** and **Tutorial** to Tier4 to document this Autoware + CARET integration workflow properly.

## Usage

(TODO)

## References

- [CARET GitHub](https://github.com/tier4/caret)
- [CARET Documentation](https://tier4.github.io/caret_doc/)

# **Add Manual Control in Autoware**

## Goal

Spawn an NPC (`vehicle.tesla.model3`) in CARLA, drive it manually with `carla_manual_control` to a desired stop position, then let the Autoware ego (`vehicle.toyota.prius`) avoid it — for evaluating the avoidance scenario.

## What didn't work — and why

Manual control was not achievable in this setup. The root cause is that **`carla_ros_bridge` and `autoware_carla_interface` are two different CARLA integrations with incompatible spawn mechanisms.**

| Aspect                            | `carla_ros_bridge`                          | `autoware_carla_interface`         |
| --------------------------------- | ------------------------------------------- | ---------------------------------- |
| Spawn mechanism                   | Provides `/carla/spawn_object` ROS service  | Calls CARLA Python API directly    |
| Auto-creates ROS topics per actor | ✅                                          | ❌                                 |
| Topic naming                      | `/carla/<role>/<sensor>` (generic)          | `/sensing/lidar/...` (Autoware-native) |

`carla_spawn_objects` (and the forked `carla_spawn_dummy_objects` package I built for this) depends on the `/carla/spawn_object` service. Since `autoware_carla_interface` does not provide it, the launch hangs after `process started` and no actor is created.

A misleading detail: `ros2 service list` still showed `/carla/spawn_object`, but this was a ghost from a previous `carla_ros_bridge` run lingering in ROS 2 discovery — the service handler was dead. Likewise `carla_manual_control` depends on `/carla/<role>/...` topics that `autoware_carla_interface` never publishes for external actors, so the pygame HUD opens but stays at zero.

## Resolution

Skipped manual control. For an avoidance evaluation, a parked NPC is enough, so the obstacle is spawned statically via the CARLA Python API directly:

```python
import carla, math

c = carla.Client('localhost', 2000); c.set_timeout(5)
w = c.get_world()
ego = next(a for a in w.get_actors()
           if a.attributes.get('role_name') == 'autoware_vehicle')

tf = ego.get_transform()
yaw = math.radians(tf.rotation.yaw)
x = tf.location.x + 20 * math.cos(yaw)
y = tf.location.y + 20 * math.sin(yaw)

bp = w.get_blueprint_library().find('vehicle.tesla.model3')
bp.set_attribute('role_name', 'obstacle_01')
w.spawn_actor(bp, carla.Transform(
    carla.Location(x=x, y=y, z=tf.location.z + 0.5),
    carla.Rotation(yaw=tf.rotation.yaw)))
```

Autoware's perception pipeline picks the actor up through the ego's LiDAR/camera, so no additional ROS topics are needed.

## Takeaways

- Don't assume CARLA integrations are interchangeable. `autoware_carla_interface` is *not* a drop-in alternative to `carla_ros_bridge` from a tooling perspective.
- ROS 2 discovery can show ghost services after stale processes. When a call hangs, verify the publisher/subscriber count, not just service/topic listings.
- If dynamic NPCs become necessary later (e.g., cut-in scenarios), the `carla_spawn_dummy_objects` package is built and ready — it just needs its spawn logic rewritten to call the CARLA Python API instead of the missing ROS service.

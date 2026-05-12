# **Synchronous vs Asynchronous I/O**

A concept that comes up constantly in evaluation, simulation, and network/device I/O. Synchronous vs Asynchronous is frequently confused with Blocking vs Non-Blocking, but they are in fact **two independent axes**, and all four combinations are valid.

!!! important
    - **Blocking / Non-Blocking** → *when control is returned* (when does the call return).
    - **Synchronous / Asynchronous** → *who handles the result* (does the caller wait on it, or does the callee notify it).

---

## **The Two Axes**

### **Blocking / Non-Blocking — When Control Is Returned**

- **Blocking**: when A calls B, A does not get control back until B's work is finished. While B runs, A *can do nothing else*.
- **Non-Blocking**: when A calls B, B returns control to A immediately. A is then *free to do other work*.

### **Synchronous / Asynchronous — Who Owns the Result**

- **Synchronous**: A checks and processes the result/completion of B *itself*. A *cares about* the result directly.
- **Asynchronous**: B notifies A of its result/completion through a callback (or similar). After the call, A *does not need to track the result*.

---

## **The Four Combinations**

### **① Sync-Blocking — Simplest, Least Efficient**

- A calls B and just waits until the return value arrives.
- The result is the return value, received directly by A.
- **Examples**: ordinary function calls, default `read()` / `recv()` syscalls, `std::cin >>`.

### **② Sync-NonBlocking — The Polling Pattern**

- The call to B returns immediately, but the result is not yet available ("returns right away, unfinished").
- A does other work, periodically asking "are you done yet?" (polling).
- Collecting the result is still A's responsibility.
- **Examples**: `read()` on an fd opened with `O_NONBLOCK`, polling loops based on `select` / `poll`.

### **③ Async-Blocking — The "Async in Form Only" Paradox**

- A registers a callback but then just blocks until the callback fires.
- "Synchronous behavior wearing async API clothing."
- Usually an anti-pattern, but it can be legitimate when you want to wait on many async I/Os at once via `select` / `epoll`.
- **Examples**: calling `.get()` / `.wait()` on a Future/Promise immediately, `async_send_request(...).get()` (rclcpp).

### **④ Async-NonBlocking — The Modern Async I/O Standard**

- The call to B returns immediately and A is free to do other work.
- When the work finishes, a callback is invoked automatically to handle the result.
- The most efficient pattern, but the hardest to follow as flow (callback hell, mitigated by async/await).
- **Examples**: Node.js `fs.readFile`, POSIX AIO, `io_uring`, JavaScript `Promise` + `await`, CUDA stream + event callback.

---

## **Quadrant Summary**

|                                       | Blocking (control held)                   | Non-Blocking (control returned immediately) |
| ------------------------------------- | ----------------------------------------- | ------------------------------------------- |
| **Synchronous** (A collects result)   | ① Sync-Blocking — just wait after call    | ② Sync-NonBlocking — check via polling      |
| **Asynchronous** (B notifies result)  | ③ Async-Blocking — register callback then wait | ④ Async-NonBlocking — register callback then do other work |

---

## **ROS / Autoware Mapping**

| Pattern                                                                          | Category          |
| -------------------------------------------------------------------------------- | ----------------- |
| `rclcpp::spin()` on a single-threaded executor, waiting for the next callback   | Sync-Blocking     |
| Periodically checking topic state with a timer                                   | Sync-NonBlocking  |
| `client->async_send_request(req).get()`                                          | Async-Blocking    |
| Subscription callback under MultiThreadedExecutor + reentrant callback group     | Async-NonBlocking |
| TensorRT `enqueueV3` → `cudaStreamSynchronize`                                   | Async-Blocking    |
| TensorRT `enqueueV3` → post-processing on a different stream                     | Async-NonBlocking |

---

## **One-Line Summary**

> **Blocking is about *whether you wait*; Sync is about *who collects the result*.** The two axes are independent, so all four combinations exist. Modern systems aim for ④ Async-NonBlocking.

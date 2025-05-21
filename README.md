# Battery-Swap Routing Optimiser – Solution Overview

## ✅ Assumptions

| Parameter               | Value / Description                         |
|------------------------|---------------------------------------------|
| Riders in snapshot     | 100 riders with random coordinates in Pune  |
| Swap stations          | 3, placed across city limits                |
| Battery consumption    | 4% SOC per km                               |
| Minimum allowed SOC    | 10%                                         |
| Swap time              | 4 minutes per rider                         |
| Max station queue      | 5 riders max in queue at a time             |
| Travel speed estimate  | 30 km/h (used to estimate time to stations) |
| SOC recharge           | SOC instantly restored to 100% after swap   |

- All timestamps are handled in UTC using ISO 8601 format (with `"Z"` suffix).
- Riders may be `idle` or `on_gig`; gig finish time is respected in schedule.
- Queues are tracked in time units, ensuring no more than 5 simultaneous overlaps.

---

## 🧠 Algorithm & Heuristic Logic

1. **Preprocessing**
   - Riders and stations are generated using mock/random data within Pune bounds.
   - Initial station queues are simulated by pushing placeholder timestamps into their queue buffer.

2. **Rider Eligibility Check**
   - For each rider, we compute effective SOC after current gig (if applicable).
   - Riders are considered for scheduling **only if their post-gig SOC may fall near the 10% threshold** (specifically `< MIN_SOC_ALLOWED + buffer`).

3. **Station Selection Heuristic**
   - For eligible riders, we find the **nearest station** where:
     - Distance can be safely traveled (without breaching SOC < 10%).
     - Rider won't increase queue count above 5 at estimated arrival time.

4. **Time Scheduling**
   - Arrival time is computed based on distance and assumed speed (30 km/h).
   - Swap start time is calculated by checking:
     - All current riders queued at that station.
     - Ensuring not more than 5 riders are scheduled in any 4-minute window.
   - If queue is full, we incrementally push the swap start time by 1 minute until a free slot is found.

5. **Output Generation**
   - For every scheduled rider, we log:
     - Timestamps for depart, arrival, swap start/end.
     - Coordinates of the return point (same as swap station).

---

## 🎯 How Objectives Are Met

| Objective                              | Approach / Guarantee |
|----------------------------------------|----------------------|
| SOC never < 10%                        | We simulate post-gig SOC and ensure chosen station is reachable with ≥ 10% SOC remaining |
| Total detour km minimized             | Nearest station satisfying SOC + queue constraints is always selected |
| Station queue never > 5 riders        | Swap start time is iteratively delayed until ≤ 5 concurrent swaps in 4-minute windows |

---

## 📈 Scalability Thoughts

- **Time Complexity**: For `n` riders and `m` stations, complexity is approximately `O(n * m)` due to nearest-station selection loop.
- **Queue Management**: Uses simple timestamp arrays and `datetime` arithmetic; can be optimized using heap queues or scheduling libraries.
- **Future Scaling**:
  - Replace queue simulation with a real-time task scheduler for live data streams.
- **Real-world extensions**:
  - Incorporate traffic delays via Maps API.
  - Integrate station capacity prediction models.
  - Add prioritization (e.g., based on urgency or gig deadline).

---

## 🔚 Summary

This solution balances simplicity, constraint satisfaction, and clarity. It produces an efficient, valid battery swap plan while ensuring SOC safety and low queue congestion. The logic is modular and scalable for real-world use in smart electric mobility logistics.


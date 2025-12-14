# Elevator System

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [Class Design](#class-design)
4. [State Management](#state-management)
5. [Multi-threading Considerations](#multi-threading-considerations)
6. [Complete Implementation](#complete-implementation)

---

## Problem Statement

Design an elevator system that can:
- Handle multiple elevators
- Process requests from different floors
- Optimize elevator movement
- Handle concurrent requests
- Support different elevator states

---

## Requirements Analysis

### Functional Requirements

1. **Elevator Operations**
   - Move up/down
   - Open/close doors
   - Stop at requested floors
   - Display current floor

2. **Request Handling**
   - Process floor requests
   - Queue requests
   - Optimize path

3. **Safety**
   - Weight limit
   - Emergency stop
   - Door safety

---

## Class Design

### Core Classes

```java
// Elevator State
public enum ElevatorState {
    IDLE,
    MOVING_UP,
    MOVING_DOWN,
    DOORS_OPEN,
    MAINTENANCE
}

// Direction
public enum Direction {
    UP,
    DOWN,
    NONE
}

// Elevator class
public class Elevator {
    private final int id;
    private int currentFloor;
    private ElevatorState state;
    private Direction direction;
    private Set<Integer> requestedFloors;
    private int maxWeight;
    private int currentWeight;
    
    public Elevator(int id, int maxWeight) {
        this.id = id;
        this.currentFloor = 0;
        this.state = ElevatorState.IDLE;
        this.direction = Direction.NONE;
        this.requestedFloors = new TreeSet<>();
        this.maxWeight = maxWeight;
        this.currentWeight = 0;
    }
    
    public synchronized void requestFloor(int floor) {
        if (floor < 0 || floor > 10) {
            throw new IllegalArgumentException("Invalid floor");
        }
        requestedFloors.add(floor);
        updateDirection();
    }
    
    public synchronized void move() {
        if (requestedFloors.isEmpty()) {
            state = ElevatorState.IDLE;
            direction = Direction.NONE;
            return;
        }
        
        if (state == ElevatorState.DOORS_OPEN) {
            closeDoors();
        }
        
        int nextFloor = getNextFloor();
        if (nextFloor > currentFloor) {
            direction = Direction.UP;
            state = ElevatorState.MOVING_UP;
            currentFloor++;
        } else if (nextFloor < currentFloor) {
            direction = Direction.DOWN;
            state = ElevatorState.MOVING_DOWN;
            currentFloor--;
        } else {
            stop();
        }
    }
    
    private int getNextFloor() {
        if (direction == Direction.UP) {
            return requestedFloors.stream()
                .filter(f -> f > currentFloor)
                .min(Integer::compare)
                .orElse(requestedFloors.stream()
                    .max(Integer::compare)
                    .orElse(currentFloor));
        } else {
            return requestedFloors.stream()
                .filter(f -> f < currentFloor)
                .max(Integer::compare)
                .orElse(requestedFloors.stream()
                    .min(Integer::compare)
                    .orElse(currentFloor));
        }
    }
    
    private void stop() {
        state = ElevatorState.DOORS_OPEN;
        requestedFloors.remove(currentFloor);
        openDoors();
    }
    
    private void openDoors() {
        System.out.println("Elevator " + id + " doors opening at floor " + currentFloor);
    }
    
    private void closeDoors() {
        System.out.println("Elevator " + id + " doors closing");
    }
    
    private void updateDirection() {
        if (requestedFloors.isEmpty()) {
            direction = Direction.NONE;
        } else if (state == ElevatorState.IDLE) {
            int nextFloor = requestedFloors.iterator().next();
            direction = nextFloor > currentFloor ? Direction.UP : Direction.DOWN;
        }
    }
}

// Elevator Controller
public class ElevatorController {
    private List<Elevator> elevators;
    
    public ElevatorController(int numberOfElevators) {
        this.elevators = new ArrayList<>();
        for (int i = 0; i < numberOfElevators; i++) {
            elevators.add(new Elevator(i + 1, 1000));
        }
    }
    
    public void requestElevator(int floor, Direction direction) {
        Elevator bestElevator = findBestElevator(floor, direction);
        bestElevator.requestFloor(floor);
    }
    
    private Elevator findBestElevator(int floor, Direction direction) {
        // Find closest idle elevator
        Elevator idle = elevators.stream()
            .filter(e -> e.getState() == ElevatorState.IDLE)
            .min(Comparator.comparingInt(e -> 
                Math.abs(e.getCurrentFloor() - floor)))
            .orElse(null);
        
        if (idle != null) {
            return idle;
        }
        
        // Find elevator moving in same direction
        return elevators.stream()
            .filter(e -> e.getDirection() == direction)
            .min(Comparator.comparingInt(e -> 
                Math.abs(e.getCurrentFloor() - floor)))
            .orElse(elevators.get(0));
    }
    
    public void start() {
        ScheduledExecutorService executor = Executors.newScheduledThreadPool(elevators.size());
        for (Elevator elevator : elevators) {
            executor.scheduleAtFixedRate(
                elevator::move,
                0,
                1,
                TimeUnit.SECONDS
            );
        }
    }
}
```

---

## State Management

### Elevator States

```
IDLE → MOVING_UP → DOORS_OPEN → IDLE
  │
  └──> MOVING_DOWN → DOORS_OPEN → IDLE
```

### State Transitions

```java
public void handleStateTransition() {
    switch (state) {
        case IDLE:
            if (!requestedFloors.isEmpty()) {
                state = determineDirection();
            }
            break;
        case MOVING_UP:
            if (currentFloor == targetFloor) {
                state = ElevatorState.DOORS_OPEN;
            }
            break;
        case DOORS_OPEN:
            if (requestedFloors.isEmpty()) {
                state = ElevatorState.IDLE;
            } else {
                state = determineDirection();
            }
            break;
    }
}
```

---

## Multi-threading Considerations

### Thread Safety

```java
public class Elevator {
    private final Object lock = new Object();
    
    public void requestFloor(int floor) {
        synchronized (lock) {
            requestedFloors.add(floor);
            updateDirection();
        }
    }
    
    public void move() {
        synchronized (lock) {
            // Move logic
        }
    }
}
```

---

## Summary

**Key Components:**
1. ✅ **Elevator:** Handles movement and state
2. ✅ **ElevatorController:** Manages multiple elevators
3. ✅ **State Management:** IDLE, MOVING, DOORS_OPEN
4. ✅ **Thread Safety:** Synchronized operations

**Next Steps:**
- Practice [ATM System](./15_ATM_SYSTEM.md)
- Learn [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- Review [Concurrency](./21_CONCURRENCY_IN_LLD.md)

---

**References:**
- [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- [Concurrency](./21_CONCURRENCY_IN_LLD.md)


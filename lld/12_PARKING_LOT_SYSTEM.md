# Parking Lot System - Complete LLD Design

## Problem Statement

Design a parking lot system that can:
1. Park vehicles (cars, motorcycles, trucks)
2. Unpark vehicles
3. Find available spots
4. Calculate parking fees
5. Support multiple parking floors
6. Handle different vehicle types with different spot sizes

---

## Requirements Analysis

### Functional Requirements
1. Park a vehicle in an available spot
2. Unpark a vehicle and calculate fee
3. Find available spots for a vehicle type
4. Display parking lot status
5. Support multiple floors
6. Support different vehicle types (Car, Motorcycle, Truck)

### Non-Functional Requirements
1. Thread-safe (multiple vehicles can park/unpark simultaneously)
2. Scalable (can add more floors/spots)
3. Efficient spot allocation
4. Clear separation of concerns

---

## Class Design

### Core Classes

```
┌─────────────────────────────────────────┐
│         ParkingLotSystem                │
│  - floors: List<ParkingFloor>           │
│  + parkVehicle(Vehicle): Ticket         │
│  + unparkVehicle(Ticket): Receipt       │
│  + getAvailableSpots(VehicleType): int  │
└─────────────────────────────────────────┘
              │
              ├─────────────────┬─────────────────┐
              ▼                 ▼                 ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │ ParkingFloor │  │   Vehicle    │  │    Ticket    │
    │ - spots      │  │ - type       │  │ - id         │
    │ - floorNum   │  │ - license    │  │ - vehicle    │
    │ + park()     │  │              │  │ - spot       │
    │ + unpark()   │  └──────────────┘  │ - entryTime  │
    └──────────────┘                    └──────────────┘
              │
              ▼
    ┌──────────────┐
    │ ParkingSpot  │
    │ - id         │
    │ - type       │
    │ - isOccupied │
    │ - vehicle    │
    └──────────────┘
```

---

## Implementation

### 1. Enums

```java
public enum VehicleType {
    MOTORCYCLE(1),
    CAR(2),
    TRUCK(3);
    
    private final int spotsRequired;
    
    VehicleType(int spotsRequired) {
        this.spotsRequired = spotsRequired;
    }
    
    public int getSpotsRequired() {
        return spotsRequired;
    }
}

public enum SpotType {
    SMALL(1),      // For motorcycles
    MEDIUM(2),     // For cars
    LARGE(3);      // For trucks
    
    private final int size;
    
    SpotType(int size) {
        this.size = size;
    }
    
    public int getSize() {
        return size;
    }
    
    public boolean canFit(VehicleType vehicleType) {
        return this.size >= vehicleType.getSpotsRequired();
    }
}

public enum SpotStatus {
    AVAILABLE,
    OCCUPIED,
    RESERVED
}
```

### 2. Vehicle Class

```java
public abstract class Vehicle {
    private String licensePlate;
    private VehicleType type;
    
    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public String getLicensePlate() {
        return licensePlate;
    }
    
    public VehicleType getType() {
        return type;
    }
}

public class Motorcycle extends Vehicle {
    public Motorcycle(String licensePlate) {
        super(licensePlate, VehicleType.MOTORCYCLE);
    }
}

public class Car extends Vehicle {
    public Car(String licensePlate) {
        super(licensePlate, VehicleType.CAR);
    }
}

public class Truck extends Vehicle {
    public Truck(String licensePlate) {
        super(licensePlate, VehicleType.TRUCK);
    }
}
```

### 3. ParkingSpot Class

```java
public class ParkingSpot {
    private String spotId;
    private SpotType type;
    private SpotStatus status;
    private Vehicle vehicle;
    private int floorNumber;
    
    public ParkingSpot(String spotId, SpotType type, int floorNumber) {
        this.spotId = spotId;
        this.type = type;
        this.floorNumber = floorNumber;
        this.status = SpotStatus.AVAILABLE;
    }
    
    public boolean isAvailable() {
        return status == SpotStatus.AVAILABLE;
    }
    
    public boolean canFitVehicle(VehicleType vehicleType) {
        return type.canFit(vehicleType) && isAvailable();
    }
    
    public void parkVehicle(Vehicle vehicle) {
        if (!canFitVehicle(vehicle.getType())) {
            throw new IllegalStateException("Spot cannot fit this vehicle");
        }
        this.vehicle = vehicle;
        this.status = SpotStatus.OCCUPIED;
    }
    
    public Vehicle unparkVehicle() {
        if (status != SpotStatus.OCCUPIED) {
            throw new IllegalStateException("Spot is not occupied");
        }
        Vehicle parkedVehicle = this.vehicle;
        this.vehicle = null;
        this.status = SpotStatus.AVAILABLE;
        return parkedVehicle;
    }
    
    // Getters
    public String getSpotId() { return spotId; }
    public SpotType getType() { return type; }
    public SpotStatus getStatus() { return status; }
    public Vehicle getVehicle() { return vehicle; }
    public int getFloorNumber() { return floorNumber; }
}
```

### 4. Ticket Class

```java
public class Ticket {
    private String ticketId;
    private Vehicle vehicle;
    private ParkingSpot spot;
    private LocalDateTime entryTime;
    
    public Ticket(String ticketId, Vehicle vehicle, ParkingSpot spot) {
        this.ticketId = ticketId;
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = LocalDateTime.now();
    }
    
    public long getParkingDurationInHours() {
        return ChronoUnit.HOURS.between(entryTime, LocalDateTime.now());
    }
    
    // Getters
    public String getTicketId() { return ticketId; }
    public Vehicle getVehicle() { return vehicle; }
    public ParkingSpot getSpot() { return spot; }
    public LocalDateTime getEntryTime() { return entryTime; }
}
```

### 5. Receipt Class

```java
public class Receipt {
    private String receiptId;
    private Ticket ticket;
    private LocalDateTime exitTime;
    private BigDecimal amount;
    
    public Receipt(String receiptId, Ticket ticket, BigDecimal amount) {
        this.receiptId = receiptId;
        this.ticket = ticket;
        this.exitTime = LocalDateTime.now();
        this.amount = amount;
    }
    
    // Getters
    public String getReceiptId() { return receiptId; }
    public Ticket getTicket() { return ticket; }
    public LocalDateTime getExitTime() { return exitTime; }
    public BigDecimal getAmount() { return amount; }
}
```

### 6. ParkingFloor Class

```java
public class ParkingFloor {
    private int floorNumber;
    private List<ParkingSpot> spots;
    private Map<SpotType, List<ParkingSpot>> spotsByType;
    
    public ParkingFloor(int floorNumber, List<ParkingSpot> spots) {
        this.floorNumber = floorNumber;
        this.spots = spots;
        this.spotsByType = new HashMap<>();
        organizeSpotsByType();
    }
    
    private void organizeSpotsByType() {
        for (ParkingSpot spot : spots) {
            spotsByType.computeIfAbsent(spot.getType(), k -> new ArrayList<>()).add(spot);
        }
    }
    
    public ParkingSpot findAvailableSpot(VehicleType vehicleType) {
        for (ParkingSpot spot : spots) {
            if (spot.canFitVehicle(vehicleType)) {
                return spot;
            }
        }
        return null;
    }
    
    public int getAvailableSpotsCount(VehicleType vehicleType) {
        int count = 0;
        for (ParkingSpot spot : spots) {
            if (spot.canFitVehicle(vehicleType)) {
                count++;
            }
        }
        return count;
    }
    
    public ParkingSpot parkVehicle(Vehicle vehicle) {
        ParkingSpot spot = findAvailableSpot(vehicle.getType());
        if (spot == null) {
            return null;
        }
        spot.parkVehicle(vehicle);
        return spot;
    }
    
    public Vehicle unparkVehicle(String spotId) {
        for (ParkingSpot spot : spots) {
            if (spot.getSpotId().equals(spotId) && spot.getStatus() == SpotStatus.OCCUPIED) {
                return spot.unparkVehicle();
            }
        }
        throw new IllegalArgumentException("Spot not found or not occupied");
    }
    
    // Getters
    public int getFloorNumber() { return floorNumber; }
    public List<ParkingSpot> getSpots() { return spots; }
}
```

### 7. PricingStrategy Interface

```java
public interface PricingStrategy {
    BigDecimal calculateFee(Ticket ticket);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private Map<VehicleType, BigDecimal> hourlyRates;
    
    public HourlyPricingStrategy() {
        hourlyRates = new HashMap<>();
        hourlyRates.put(VehicleType.MOTORCYCLE, new BigDecimal("5"));
        hourlyRates.put(VehicleType.CAR, new BigDecimal("10"));
        hourlyRates.put(VehicleType.TRUCK, new BigDecimal("20"));
    }
    
    @Override
    public BigDecimal calculateFee(Ticket ticket) {
        long hours = ticket.getParkingDurationInHours();
        if (hours < 1) hours = 1; // Minimum 1 hour
        
        BigDecimal hourlyRate = hourlyRates.get(ticket.getVehicle().getType());
        return hourlyRate.multiply(new BigDecimal(hours));
    }
}
```

### 8. ParkingLotSystem Class

```java
public class ParkingLotSystem {
    private List<ParkingFloor> floors;
    private Map<String, Ticket> activeTickets;
    private PricingStrategy pricingStrategy;
    private static final AtomicInteger ticketCounter = new AtomicInteger(1);
    
    public ParkingLotSystem(List<ParkingFloor> floors, PricingStrategy pricingStrategy) {
        this.floors = floors;
        this.activeTickets = new ConcurrentHashMap<>();
        this.pricingStrategy = pricingStrategy;
    }
    
    public Ticket parkVehicle(Vehicle vehicle) {
        // Find available spot
        ParkingSpot spot = findAvailableSpot(vehicle.getType());
        if (spot == null) {
            throw new IllegalStateException("No available spots for " + vehicle.getType());
        }
        
        // Park vehicle
        spot.parkVehicle(vehicle);
        
        // Generate ticket
        String ticketId = "T" + ticketCounter.getAndIncrement();
        Ticket ticket = new Ticket(ticketId, vehicle, spot);
        activeTickets.put(ticketId, ticket);
        
        System.out.println("Vehicle " + vehicle.getLicensePlate() + 
                          " parked at spot " + spot.getSpotId() + 
                          " on floor " + spot.getFloorNumber());
        
        return ticket;
    }
    
    public Receipt unparkVehicle(String ticketId) {
        Ticket ticket = activeTickets.remove(ticketId);
        if (ticket == null) {
            throw new IllegalArgumentException("Invalid ticket ID");
        }
        
        // Unpark vehicle
        Vehicle vehicle = ticket.getSpot().unparkVehicle();
        
        // Calculate fee
        BigDecimal fee = pricingStrategy.calculateFee(ticket);
        
        // Generate receipt
        String receiptId = "R" + ticketCounter.getAndIncrement();
        Receipt receipt = new Receipt(receiptId, ticket, fee);
        
        System.out.println("Vehicle " + vehicle.getLicensePlate() + 
                          " unparked. Fee: $" + fee);
        
        return receipt;
    }
    
    private ParkingSpot findAvailableSpot(VehicleType vehicleType) {
        for (ParkingFloor floor : floors) {
            ParkingSpot spot = floor.findAvailableSpot(vehicleType);
            if (spot != null) {
                return spot;
            }
        }
        return null;
    }
    
    public int getAvailableSpotsCount(VehicleType vehicleType) {
        return floors.stream()
            .mapToInt(floor -> floor.getAvailableSpotsCount(vehicleType))
            .sum();
    }
    
    public void displayStatus() {
        System.out.println("\n=== Parking Lot Status ===");
        for (ParkingFloor floor : floors) {
            System.out.println("Floor " + floor.getFloorNumber() + ":");
            for (ParkingSpot spot : floor.getSpots()) {
                String status = spot.isAvailable() ? "Available" : 
                               "Occupied by " + spot.getVehicle().getLicensePlate();
                System.out.println("  Spot " + spot.getSpotId() + " (" + 
                                 spot.getType() + "): " + status);
            }
        }
    }
}
```

### 9. Main Class (Usage Example)

```java
public class ParkingLotDemo {
    public static void main(String[] args) {
        // Create parking spots
        List<ParkingSpot> floor1Spots = Arrays.asList(
            new ParkingSpot("F1-S1", SpotType.SMALL, 1),
            new ParkingSpot("F1-S2", SpotType.SMALL, 1),
            new ParkingSpot("F1-M1", SpotType.MEDIUM, 1),
            new ParkingSpot("F1-M2", SpotType.MEDIUM, 1),
            new ParkingSpot("F1-L1", SpotType.LARGE, 1)
        );
        
        List<ParkingSpot> floor2Spots = Arrays.asList(
            new ParkingSpot("F2-S1", SpotType.SMALL, 2),
            new ParkingSpot("F2-M1", SpotType.MEDIUM, 2),
            new ParkingSpot("F2-M2", SpotType.MEDIUM, 2),
            new ParkingSpot("F2-L1", SpotType.LARGE, 2)
        );
        
        // Create floors
        List<ParkingFloor> floors = Arrays.asList(
            new ParkingFloor(1, floor1Spots),
            new ParkingFloor(2, floor2Spots)
        );
        
        // Create parking lot system
        PricingStrategy pricingStrategy = new HourlyPricingStrategy();
        ParkingLotSystem parkingLot = new ParkingLotSystem(floors, pricingStrategy);
        
        // Park vehicles
        Vehicle car1 = new Car("ABC123");
        Vehicle motorcycle1 = new Motorcycle("XYZ789");
        Vehicle truck1 = new Truck("TRUCK1");
        
        Ticket ticket1 = parkingLot.parkVehicle(car1);
        Ticket ticket2 = parkingLot.parkVehicle(motorcycle1);
        Ticket ticket3 = parkingLot.parkVehicle(truck1);
        
        // Display status
        parkingLot.displayStatus();
        
        // Check available spots
        System.out.println("\nAvailable spots for CAR: " + 
                         parkingLot.getAvailableSpotsCount(VehicleType.CAR));
        
        // Unpark vehicle
        Thread.sleep(2000); // Simulate parking duration
        Receipt receipt1 = parkingLot.unparkVehicle(ticket1.getTicketId());
        System.out.println("Receipt: " + receipt1.getReceiptId() + 
                         ", Amount: $" + receipt1.getAmount());
        
        // Display status again
        parkingLot.displayStatus();
    }
}
```

---

## Design Patterns Used

### 1. Strategy Pattern
- `PricingStrategy` interface with different implementations
- Easy to add new pricing models (flat rate, dynamic pricing, etc.)

### 2. Factory Pattern (can be added)
- `VehicleFactory` to create vehicles based on type

### 3. Singleton Pattern (can be added)
- `ParkingLotSystem` could be singleton if only one instance needed

---

## SOLID Principles Applied

1. **Single Responsibility**: Each class has one clear responsibility
2. **Open/Closed**: Easy to add new vehicle types or pricing strategies
3. **Liskov Substitution**: All vehicles can be substituted
4. **Interface Segregation**: Small, focused interfaces
5. **Dependency Inversion**: Depends on `PricingStrategy` abstraction

---

## Extensions

### 1. Add Reservation System
```java
public interface ReservationService {
    void reserveSpot(String spotId, LocalDateTime startTime);
    void cancelReservation(String reservationId);
}
```

### 2. Add Multiple Pricing Strategies
```java
public class DynamicPricingStrategy implements PricingStrategy {
    // Higher rates during peak hours
}
```

### 3. Add Payment Processing
```java
public interface PaymentProcessor {
    PaymentResult processPayment(BigDecimal amount, PaymentMethod method);
}
```

### 4. Add Notifications
```java
public interface NotificationService {
    void sendNotification(String recipient, String message);
}
```

---

## Testing Considerations

```java
@Test
public void testParkVehicle() {
    // Test parking a vehicle
}

@Test
public void testUnparkVehicle() {
    // Test unparking and fee calculation
}

@Test
public void testNoAvailableSpots() {
    // Test when parking lot is full
}

@Test
public void testConcurrentParking() {
    // Test thread safety
}
```

---

## Key Takeaways

1. **Clear Class Hierarchy**: Vehicle → Car/Motorcycle/Truck
2. **Separation of Concerns**: Separate parking logic from pricing
3. **Extensibility**: Easy to add new vehicle types or pricing strategies
4. **Thread Safety**: Use ConcurrentHashMap for active tickets
5. **SOLID Principles**: Applied throughout the design

---

## Next Steps

1. Practice [Library Management System](./13_LIBRARY_MANAGEMENT_SYSTEM.md)
2. Study [Elevator System](./14_ELEVATOR_SYSTEM.md)
3. Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)


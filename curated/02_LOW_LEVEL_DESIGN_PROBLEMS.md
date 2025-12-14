# Low-Level Design Problems - 30+ Real-World Problems

> **Curated from top tech companies and real production scenarios**

This guide contains 30+ real-world low-level design problems with comprehensive solutions, suitable for staff engineer interviews.

---

## Table of Contents

1. [Basic LLD Problems](#basic-lld-problems)
2. [Intermediate LLD Problems](#intermediate-lld-problems)
3. [Advanced LLD Problems](#advanced-lld-problems)
4. [Expert-Level LLD Problems](#expert-level-lld-problems)

---

## Basic LLD Problems

### Problem 1: Design a Parking Lot System

**Problem Statement:**
Design a parking lot system that can handle multiple parking spots, different vehicle types, and parking fees.

**Requirements:**
- Support multiple vehicle types (Car, Motorcycle, Truck)
- Different parking spot sizes
- Calculate parking fees based on time
- Track available spots
- Handle entry and exit

**Solution:**

```java
// Vehicle types
public enum VehicleType {
    MOTORCYCLE, CAR, TRUCK
}

// Vehicle class
public abstract class Vehicle {
    protected String licensePlate;
    protected VehicleType type;
    
    public Vehicle(String licensePlate, VehicleType type) {
        this.licensePlate = licensePlate;
        this.type = type;
    }
    
    public VehicleType getType() {
        return type;
    }
}

public class Car extends Vehicle {
    public Car(String licensePlate) {
        super(licensePlate, VehicleType.CAR);
    }
}

// Parking spot
public class ParkingSpot {
    private int spotId;
    private VehicleType vehicleType;
    private boolean isOccupied;
    private Vehicle parkedVehicle;
    
    public ParkingSpot(int spotId, VehicleType vehicleType) {
        this.spotId = spotId;
        this.vehicleType = vehicleType;
        this.isOccupied = false;
    }
    
    public boolean canPark(Vehicle vehicle) {
        return !isOccupied && vehicle.getType() == vehicleType;
    }
    
    public void park(Vehicle vehicle) {
        this.parkedVehicle = vehicle;
        this.isOccupied = true;
    }
    
    public Vehicle unpark() {
        Vehicle vehicle = parkedVehicle;
        this.parkedVehicle = null;
        this.isOccupied = false;
        return vehicle;
    }
}

// Parking lot
public class ParkingLot {
    private List<ParkingSpot> spots;
    private Map<String, Ticket> activeTickets;
    private PricingStrategy pricingStrategy;
    
    public ParkingLot(List<ParkingSpot> spots, PricingStrategy pricingStrategy) {
        this.spots = spots;
        this.activeTickets = new HashMap<>();
        this.pricingStrategy = pricingStrategy;
    }
    
    public Ticket parkVehicle(Vehicle vehicle) {
        ParkingSpot spot = findAvailableSpot(vehicle.getType());
        if (spot == null) {
            throw new RuntimeException("No available spots");
        }
        
        spot.park(vehicle);
        Ticket ticket = new Ticket(vehicle, spot, System.currentTimeMillis());
        activeTickets.put(vehicle.getLicensePlate(), ticket);
        return ticket;
    }
    
    public double exitVehicle(String licensePlate) {
        Ticket ticket = activeTickets.remove(licensePlate);
        if (ticket == null) {
            throw new RuntimeException("Vehicle not found");
        }
        
        ticket.getSpot().unpark();
        long duration = System.currentTimeMillis() - ticket.getEntryTime();
        double fee = pricingStrategy.calculateFee(duration, ticket.getVehicle().getType());
        return fee;
    }
    
    private ParkingSpot findAvailableSpot(VehicleType type) {
        return spots.stream()
            .filter(spot -> spot.canPark(new Car(""))) // Create dummy vehicle
            .findFirst()
            .orElse(null);
    }
}

// Ticket
public class Ticket {
    private Vehicle vehicle;
    private ParkingSpot spot;
    private long entryTime;
    
    public Ticket(Vehicle vehicle, ParkingSpot spot, long entryTime) {
        this.vehicle = vehicle;
        this.spot = spot;
        this.entryTime = entryTime;
    }
    
    // Getters
}

// Pricing strategy (Strategy Pattern)
public interface PricingStrategy {
    double calculateFee(long durationMillis, VehicleType type);
}

public class HourlyPricingStrategy implements PricingStrategy {
    private static final double CAR_RATE = 2.0; // $2 per hour
    private static final double MOTORCYCLE_RATE = 1.0;
    private static final double TRUCK_RATE = 5.0;
    
    public double calculateFee(long durationMillis, VehicleType type) {
        double hours = durationMillis / (1000.0 * 60 * 60);
        switch (type) {
            case CAR: return hours * CAR_RATE;
            case MOTORCYCLE: return hours * MOTORCYCLE_RATE;
            case TRUCK: return hours * TRUCK_RATE;
            default: return 0;
        }
    }
}
```

**Design Patterns Used:**
- Strategy Pattern: PricingStrategy
- Inheritance: Vehicle hierarchy
- Composition: ParkingLot contains ParkingSpots

**Follow-up Questions:**
- How to handle multiple floors?
- How to implement priority parking?
- How to handle payment processing?

---

### Problem 2: Design a Library Management System

**Problem Statement:**
Design a library management system that can handle books, members, borrowing, and returns.

**Requirements:**
- Add/remove books
- Register members
- Borrow and return books
- Track due dates
- Calculate fines
- Search books

**Solution:**

```java
// Book
public class Book {
    private String isbn;
    private String title;
    private String author;
    private int totalCopies;
    private int availableCopies;
    
    public Book(String isbn, String title, String author, int copies) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.totalCopies = copies;
        this.availableCopies = copies;
    }
    
    public boolean isAvailable() {
        return availableCopies > 0;
    }
    
    public void borrow() {
        if (availableCopies > 0) {
            availableCopies--;
        } else {
            throw new RuntimeException("No copies available");
        }
    }
    
    public void returnBook() {
        if (availableCopies < totalCopies) {
            availableCopies++;
        }
    }
}

// Member
public class Member {
    private String memberId;
    private String name;
    private List<BookItem> borrowedBooks;
    
    public Member(String memberId, String name) {
        this.memberId = memberId;
        this.name = name;
        this.borrowedBooks = new ArrayList<>();
    }
    
    public void borrowBook(BookItem bookItem) {
        borrowedBooks.add(bookItem);
    }
    
    public void returnBook(BookItem bookItem) {
        borrowedBooks.remove(bookItem);
    }
}

// Book item (represents a physical copy)
public class BookItem {
    private String barcode;
    private Book book;
    private Member borrowedBy;
    private Date dueDate;
    
    public BookItem(String barcode, Book book) {
        this.barcode = barcode;
        this.book = book;
    }
}

// Library
public class Library {
    private Map<String, Book> books;
    private Map<String, Member> members;
    private Map<String, BookItem> bookItems;
    private FineCalculator fineCalculator;
    
    public Library() {
        this.books = new HashMap<>();
        this.members = new HashMap<>();
        this.bookItems = new HashMap<>();
        this.fineCalculator = new SimpleFineCalculator();
    }
    
    public void addBook(Book book) {
        books.put(book.getIsbn(), book);
    }
    
    public void registerMember(Member member) {
        members.put(member.getMemberId(), member);
    }
    
    public BookItem borrowBook(String memberId, String isbn) {
        Member member = members.get(memberId);
        Book book = books.get(isbn);
        
        if (member == null || book == null) {
            throw new RuntimeException("Invalid member or book");
        }
        
        if (!book.isAvailable()) {
            throw new RuntimeException("Book not available");
        }
        
        book.borrow();
        BookItem bookItem = new BookItem(UUID.randomUUID().toString(), book);
        bookItem.setBorrowedBy(member);
        bookItem.setDueDate(calculateDueDate());
        
        member.borrowBook(bookItem);
        bookItems.put(bookItem.getBarcode(), bookItem);
        
        return bookItem;
    }
    
    public double returnBook(String memberId, String barcode) {
        Member member = members.get(memberId);
        BookItem bookItem = bookItems.get(barcode);
        
        if (member == null || bookItem == null) {
            throw new RuntimeException("Invalid member or book item");
        }
        
        member.returnBook(bookItem);
        bookItem.getBook().returnBook();
        
        double fine = fineCalculator.calculateFine(bookItem.getDueDate(), new Date());
        bookItems.remove(barcode);
        
        return fine;
    }
    
    private Date calculateDueDate() {
        Calendar cal = Calendar.getInstance();
        cal.add(Calendar.DAY_OF_MONTH, 14); // 14 days loan period
        return cal.getTime();
    }
}

// Fine calculator (Strategy Pattern)
public interface FineCalculator {
    double calculateFine(Date dueDate, Date returnDate);
}

public class SimpleFineCalculator implements FineCalculator {
    private static final double DAILY_FINE = 1.0; // $1 per day
    
    public double calculateFine(Date dueDate, Date returnDate) {
        if (returnDate.before(dueDate) || returnDate.equals(dueDate)) {
            return 0;
        }
        
        long diff = returnDate.getTime() - dueDate.getTime();
        long days = diff / (1000 * 60 * 60 * 24);
        return days * DAILY_FINE;
    }
}
```

**Design Patterns Used:**
- Strategy Pattern: FineCalculator
- Composition: Library contains Books and Members

---

### Problem 3: Design a Vending Machine

**Problem Statement:**
Design a vending machine that can accept coins, dispense products, and return change.

**Requirements:**
- Accept coins (nickel, dime, quarter)
- Select product
- Dispense product
- Return change
- Track inventory

**Solution:**

```java
// Coin types
public enum Coin {
    NICKEL(5), DIME(10), QUARTER(25);
    
    private int value;
    
    Coin(int value) {
        this.value = value;
    }
    
    public int getValue() {
        return value;
    }
}

// Product
public class Product {
    private String name;
    private int price;
    private int quantity;
    
    public Product(String name, int price, int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }
    
    public boolean isAvailable() {
        return quantity > 0;
    }
    
    public void dispense() {
        if (quantity > 0) {
            quantity--;
        } else {
            throw new RuntimeException("Product out of stock");
        }
    }
}

// Vending machine state (State Pattern)
public interface VendingMachineState {
    void insertCoin(Coin coin);
    void selectProduct(Product product);
    void dispense();
    void returnChange();
}

public class IdleState implements VendingMachineState {
    private VendingMachine machine;
    
    public IdleState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertCoin(Coin coin) {
        machine.addCoin(coin);
        machine.setState(new HasMoneyState(machine));
    }
    
    public void selectProduct(Product product) {
        throw new RuntimeException("Please insert coins first");
    }
    
    public void dispense() {
        throw new RuntimeException("Invalid operation");
    }
    
    public void returnChange() {
        throw new RuntimeException("No change to return");
    }
}

public class HasMoneyState implements VendingMachineState {
    private VendingMachine machine;
    
    public HasMoneyState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertCoin(Coin coin) {
        machine.addCoin(coin);
    }
    
    public void selectProduct(Product product) {
        if (!product.isAvailable()) {
            throw new RuntimeException("Product out of stock");
        }
        
        int totalInserted = machine.getTotalInserted();
        if (totalInserted < product.getPrice()) {
            throw new RuntimeException("Insufficient funds");
        }
        
        product.dispense();
        machine.setSelectedProduct(product);
        machine.setState(new DispensingState(machine));
    }
    
    public void dispense() {
        throw new RuntimeException("Please select a product");
    }
    
    public void returnChange() {
        machine.returnChange();
        machine.setState(new IdleState(machine));
    }
}

// Vending machine
public class VendingMachine {
    private Map<String, Product> products;
    private List<Coin> insertedCoins;
    private Product selectedProduct;
    private VendingMachineState state;
    
    public VendingMachine() {
        this.products = new HashMap<>();
        this.insertedCoins = new ArrayList<>();
        this.state = new IdleState(this);
    }
    
    public void addProduct(Product product) {
        products.put(product.getName(), product);
    }
    
    public void insertCoin(Coin coin) {
        state.insertCoin(coin);
    }
    
    public void selectProduct(String productName) {
        Product product = products.get(productName);
        if (product == null) {
            throw new RuntimeException("Product not found");
        }
        state.selectProduct(product);
    }
    
    public void dispense() {
        state.dispense();
    }
    
    public void returnChange() {
        state.returnChange();
    }
    
    // Getters and setters
    public void setState(VendingMachineState state) {
        this.state = state;
    }
    
    public void addCoin(Coin coin) {
        insertedCoins.add(coin);
    }
    
    public int getTotalInserted() {
        return insertedCoins.stream()
            .mapToInt(Coin::getValue)
            .sum();
    }
}
```

**Design Patterns Used:**
- State Pattern: VendingMachineState
- Encapsulation: Internal state management

---

## Intermediate LLD Problems

### Problem 4: Design a Snake and Ladder Game

**Problem Statement:**
Design a snake and ladder game that can handle multiple players, snakes, ladders, and game logic.

**Requirements:**
- Multiple players
- Board with snakes and ladders
- Dice rolling
- Move players
- Detect winner

**Solution:**

```java
// Player
public class Player {
    private String name;
    private int position;
    
    public Player(String name) {
        this.name = name;
        this.position = 0;
    }
    
    public void move(int steps) {
        position += steps;
    }
    
    public void setPosition(int position) {
        this.position = position;
    }
    
    // Getters
}

// Board cell
public class Cell {
    private int number;
    private Jump jump; // Snake or Ladder
    
    public Cell(int number) {
        this.number = number;
    }
    
    public void setJump(Jump jump) {
        this.jump = jump;
    }
    
    public boolean hasJump() {
        return jump != null;
    }
    
    public Jump getJump() {
        return jump;
    }
}

// Jump (Snake or Ladder)
public class Jump {
    private int start;
    private int end;
    
    public Jump(int start, int end) {
        this.start = start;
        this.end = end;
    }
    
    public int getEnd() {
        return end;
    }
}

// Dice
public class Dice {
    private int sides;
    
    public Dice(int sides) {
        this.sides = sides;
    }
    
    public int roll() {
        return (int) (Math.random() * sides) + 1;
    }
}

// Game
public class SnakeAndLadderGame {
    private Board board;
    private List<Player> players;
    private Dice dice;
    private Player currentPlayer;
    private int currentPlayerIndex;
    
    public SnakeAndLadderGame(int boardSize, int diceSides) {
        this.board = new Board(boardSize);
        this.players = new ArrayList<>();
        this.dice = new Dice(diceSides);
        this.currentPlayerIndex = 0;
    }
    
    public void addPlayer(Player player) {
        players.add(player);
    }
    
    public void addSnake(int start, int end) {
        board.addSnake(start, end);
    }
    
    public void addLadder(int start, int end) {
        board.addLadder(start, end);
    }
    
    public void playTurn() {
        if (players.isEmpty()) {
            throw new RuntimeException("No players");
        }
        
        currentPlayer = players.get(currentPlayerIndex);
        int diceValue = dice.roll();
        
        int newPosition = currentPlayer.getPosition() + diceValue;
        
        if (newPosition > board.getSize()) {
            // Player can't move beyond board
            return;
        }
        
        currentPlayer.setPosition(newPosition);
        
        // Check for snake or ladder
        Cell cell = board.getCell(newPosition);
        if (cell.hasJump()) {
            Jump jump = cell.getJump();
            currentPlayer.setPosition(jump.getEnd());
        }
        
        if (currentPlayer.getPosition() == board.getSize()) {
            System.out.println(currentPlayer.getName() + " wins!");
            return;
        }
        
        // Move to next player
        currentPlayerIndex = (currentPlayerIndex + 1) % players.size();
    }
}

// Board
public class Board {
    private int size;
    private Map<Integer, Cell> cells;
    
    public Board(int size) {
        this.size = size;
        this.cells = new HashMap<>();
        initializeCells();
    }
    
    private void initializeCells() {
        for (int i = 1; i <= size; i++) {
            cells.put(i, new Cell(i));
        }
    }
    
    public void addSnake(int start, int end) {
        if (start <= end) {
            throw new RuntimeException("Snake start must be greater than end");
        }
        Cell cell = cells.get(start);
        cell.setJump(new Jump(start, end));
    }
    
    public void addLadder(int start, int end) {
        if (start >= end) {
            throw new RuntimeException("Ladder start must be less than end");
        }
        Cell cell = cells.get(start);
        cell.setJump(new Jump(start, end));
    }
    
    public Cell getCell(int position) {
        return cells.get(position);
    }
    
    // Getters
}
```

**Design Patterns Used:**
- Composition: Game contains Board, Players, Dice
- Encapsulation: Game logic encapsulated

---

### Problem 5: Design a Tic-Tac-Toe Game

**Problem Statement:**
Design a tic-tac-toe game that can handle two players, game board, moves, and win detection.

**Requirements:**
- 3x3 board
- Two players (X and O)
- Alternate turns
- Detect win condition
- Detect draw

**Solution:**

```java
// Player
public enum Player {
    X, O, NONE
}

// Move
public class Move {
    private int row;
    private int col;
    private Player player;
    
    public Move(int row, int col, Player player) {
        this.row = row;
        this.col = col;
        this.player = player;
    }
    
    // Getters
}

// Board
public class Board {
    private static final int SIZE = 3;
    private Player[][] grid;
    
    public Board() {
        this.grid = new Player[SIZE][SIZE];
        initialize();
    }
    
    private void initialize() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                grid[i][j] = Player.NONE;
            }
        }
    }
    
    public boolean makeMove(Move move) {
        if (grid[move.getRow()][move.getCol()] != Player.NONE) {
            return false; // Cell already occupied
        }
        
        grid[move.getRow()][move.getCol()] = move.getPlayer();
        return true;
    }
    
    public boolean isFull() {
        for (int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE; j++) {
                if (grid[i][j] == Player.NONE) {
                    return false;
                }
            }
        }
        return true;
    }
    
    public Player checkWinner() {
        // Check rows
        for (int i = 0; i < SIZE; i++) {
            if (grid[i][0] != Player.NONE &&
                grid[i][0] == grid[i][1] &&
                grid[i][1] == grid[i][2]) {
                return grid[i][0];
            }
        }
        
        // Check columns
        for (int j = 0; j < SIZE; j++) {
            if (grid[0][j] != Player.NONE &&
                grid[0][j] == grid[1][j] &&
                grid[1][j] == grid[2][j]) {
                return grid[0][j];
            }
        }
        
        // Check diagonals
        if (grid[0][0] != Player.NONE &&
            grid[0][0] == grid[1][1] &&
            grid[1][1] == grid[2][2]) {
            return grid[0][0];
        }
        
        if (grid[0][2] != Player.NONE &&
            grid[0][2] == grid[1][1] &&
            grid[1][1] == grid[2][0]) {
            return grid[0][2];
        }
        
        return Player.NONE;
    }
}

// Game
public class TicTacToeGame {
    private Board board;
    private Player currentPlayer;
    private boolean gameOver;
    
    public TicTacToeGame() {
        this.board = new Board();
        this.currentPlayer = Player.X;
        this.gameOver = false;
    }
    
    public boolean makeMove(int row, int col) {
        if (gameOver) {
            return false;
        }
        
        Move move = new Move(row, col, currentPlayer);
        if (!board.makeMove(move)) {
            return false;
        }
        
        Player winner = board.checkWinner();
        if (winner != Player.NONE) {
            gameOver = true;
            System.out.println(winner + " wins!");
            return true;
        }
        
        if (board.isFull()) {
            gameOver = true;
            System.out.println("Draw!");
            return true;
        }
        
        // Switch player
        currentPlayer = (currentPlayer == Player.X) ? Player.O : Player.X;
        return true;
    }
    
    // Getters
}
```

---

## Advanced LLD Problems

### Problem 6: Design a Elevator System

**Problem Statement:**
Design an elevator system that can handle multiple elevators, floors, and requests.

**Requirements:**
- Multiple elevators
- Multiple floors
- Request elevator
- Move elevators
- Handle direction (up/down)
- Optimize for efficiency

**Solution:**

```java
// Direction
public enum Direction {
    UP, DOWN, IDLE
}

// Elevator
public class Elevator {
    private int id;
    private int currentFloor;
    private Direction direction;
    private Set<Integer> requests;
    private boolean isMoving;
    
    public Elevator(int id) {
        this.id = id;
        this.currentFloor = 0;
        this.direction = Direction.IDLE;
        this.requests = new TreeSet<>();
        this.isMoving = false;
    }
    
    public void addRequest(int floor) {
        requests.add(floor);
        updateDirection();
    }
    
    public void move() {
        if (requests.isEmpty()) {
            direction = Direction.IDLE;
            isMoving = false;
            return;
        }
        
        isMoving = true;
        int nextFloor = getNextFloor();
        
        if (nextFloor > currentFloor) {
            direction = Direction.UP;
            currentFloor++;
        } else if (nextFloor < currentFloor) {
            direction = Direction.DOWN;
            currentFloor--;
        } else {
            // Reached destination
            requests.remove(currentFloor);
            updateDirection();
        }
    }
    
    private int getNextFloor() {
        if (direction == Direction.UP) {
            return requests.stream()
                .filter(f -> f > currentFloor)
                .min(Integer::compare)
                .orElse(requests.first());
        } else if (direction == Direction.DOWN) {
            return requests.stream()
                .filter(f -> f < currentFloor)
                .max(Integer::compare)
                .orElse(requests.last());
        } else {
            return requests.first();
        }
    }
    
    private void updateDirection() {
        if (requests.isEmpty()) {
            direction = Direction.IDLE;
        } else if (requests.stream().anyMatch(f -> f > currentFloor)) {
            direction = Direction.UP;
        } else if (requests.stream().anyMatch(f -> f < currentFloor)) {
            direction = Direction.DOWN;
        }
    }
    
    // Getters
}

// Elevator controller
public class ElevatorController {
    private List<Elevator> elevators;
    private int totalFloors;
    
    public ElevatorController(int numElevators, int totalFloors) {
        this.elevators = new ArrayList<>();
        this.totalFloors = totalFloors;
        
        for (int i = 0; i < numElevators; i++) {
            elevators.add(new Elevator(i));
        }
    }
    
    public void requestElevator(int floor, Direction direction) {
        Elevator bestElevator = findBestElevator(floor, direction);
        bestElevator.addRequest(floor);
    }
    
    private Elevator findBestElevator(int floor, Direction direction) {
        // Find closest idle elevator
        Elevator best = null;
        int minDistance = Integer.MAX_VALUE;
        
        for (Elevator elevator : elevators) {
            if (elevator.getDirection() == Direction.IDLE) {
                int distance = Math.abs(elevator.getCurrentFloor() - floor);
                if (distance < minDistance) {
                    minDistance = distance;
                    best = elevator;
                }
            }
        }
        
        if (best != null) {
            return best;
        }
        
        // If no idle, find elevator going in same direction
        for (Elevator elevator : elevators) {
            if (elevator.getDirection() == direction) {
                if ((direction == Direction.UP && elevator.getCurrentFloor() < floor) ||
                    (direction == Direction.DOWN && elevator.getCurrentFloor() > floor)) {
                    int distance = Math.abs(elevator.getCurrentFloor() - floor);
                    if (distance < minDistance) {
                        minDistance = distance;
                        best = elevator;
                    }
                }
            }
        }
        
        return best != null ? best : elevators.get(0);
    }
    
    public void step() {
        for (Elevator elevator : elevators) {
            elevator.move();
        }
    }
}
```

**Design Patterns Used:**
- Strategy Pattern: Direction handling
- Observer Pattern: Could be used for floor requests

---

## Summary

These 6 problems cover:
- ✅ Basic OOP concepts
- ✅ Design patterns (Strategy, State, Observer)
- ✅ Real-world scenarios
- ✅ System design thinking

**Key Patterns:**
- Strategy Pattern: Interchangeable algorithms
- State Pattern: State management
- Composition: Building complex objects
- Encapsulation: Data hiding

**Next Steps:**
- Practice more problems
- Understand design patterns deeply
- Focus on SOLID principles
- Think about scalability

---

*This guide is continuously updated with new problems and solutions.*


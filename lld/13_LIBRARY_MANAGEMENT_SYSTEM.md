# Library Management System

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [Class Design](#class-design)
4. [Detailed Implementation](#detailed-implementation)
5. [UML Diagrams](#uml-diagrams)

---

## Problem Statement

Design a library management system that can:
- Manage books (add, remove, search)
- Manage members (register, update)
- Handle book lending and returning
- Track fines for overdue books
- Generate reports

---

## Requirements Analysis

### Functional Requirements

1. **Book Management**
   - Add/remove books
   - Search books
   - Track availability

2. **Member Management**
   - Register members
   - Update member info
   - Track borrowing history

3. **Lending System**
   - Issue books
   - Return books
   - Calculate fines

4. **Reports**
   - Books available
   - Overdue books
   - Member borrowing history

---

## Class Design

### Core Classes

```java
// Book class
public class Book {
    private final String isbn;
    private String title;
    private String author;
    private boolean isAvailable;
    private LocalDate dueDate;
    
    public Book(String isbn, String title, String author) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.isAvailable = true;
    }
    
    public void borrow() {
        if (!isAvailable) {
            throw new IllegalStateException("Book not available");
        }
        this.isAvailable = false;
        this.dueDate = LocalDate.now().plusDays(14);
    }
    
    public void returnBook() {
        this.isAvailable = true;
        this.dueDate = null;
    }
    
    public boolean isOverdue() {
        return dueDate != null && LocalDate.now().isAfter(dueDate);
    }
}

// Member class
public class Member {
    private final String memberId;
    private String name;
    private String email;
    private List<Book> borrowedBooks;
    
    public Member(String memberId, String name, String email) {
        this.memberId = memberId;
        this.name = name;
        this.email = email;
        this.borrowedBooks = new ArrayList<>();
    }
    
    public void borrowBook(Book book) {
        if (borrowedBooks.size() >= 5) {
            throw new IllegalStateException("Maximum 5 books allowed");
        }
        book.borrow();
        borrowedBooks.add(book);
    }
    
    public void returnBook(Book book) {
        borrowedBooks.remove(book);
        book.returnBook();
    }
}

// Library class
public class Library {
    private Map<String, Book> books;
    private Map<String, Member> members;
    private FineCalculator fineCalculator;
    
    public Library() {
        this.books = new HashMap<>();
        this.members = new HashMap<>();
        this.fineCalculator = new FineCalculator();
    }
    
    public void addBook(Book book) {
        books.put(book.getIsbn(), book);
    }
    
    public void registerMember(Member member) {
        members.put(member.getMemberId(), member);
    }
    
    public void issueBook(String memberId, String isbn) {
        Member member = members.get(memberId);
        Book book = books.get(isbn);
        
        if (member == null) {
            throw new IllegalArgumentException("Member not found");
        }
        if (book == null) {
            throw new IllegalArgumentException("Book not found");
        }
        
        member.borrowBook(book);
    }
    
    public double returnBook(String memberId, String isbn) {
        Member member = members.get(memberId);
        Book book = books.get(isbn);
        
        member.returnBook(book);
        
        if (book.isOverdue()) {
            return fineCalculator.calculateFine(book);
        }
        return 0.0;
    }
}

// Fine Calculator
public class FineCalculator {
    private static final double DAILY_FINE = 1.0;
    
    public double calculateFine(Book book) {
        if (!book.isOverdue()) {
            return 0.0;
        }
        long daysOverdue = ChronoUnit.DAYS.between(
            book.getDueDate(), 
            LocalDate.now()
        );
        return daysOverdue * DAILY_FINE;
    }
}
```

---

## UML Class Diagram

```
┌─────────────────────┐
│      Library        │
├─────────────────────┤
│   - books: Map      │
│   - members: Map    │
├─────────────────────┤
│   + addBook()       │
│   + registerMember()│
│   + issueBook()     │
│   + returnBook()    │
└──────────┬──────────┘
           │
    ┌──────┴──────┐
    │             │
    ▼             ▼
┌────────┐  ┌──────────┐
│  Book  │  │  Member  │
└────────┘  └────┬─────┘
                 │
                 │ *
                 │
                 ▼
            ┌────────┐
            │  Book  │
            └────────┘
```

---

## Summary

**Key Classes:**
1. ✅ **Book:** Represents a book
2. ✅ **Member:** Represents a library member
3. ✅ **Library:** Manages books and members
4. ✅ **FineCalculator:** Calculates fines

**Next Steps:**
- Practice [Elevator System](./14_ELEVATOR_SYSTEM.md)
- Learn [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- Review [Class Design](./04_CLASS_DESIGN.md)

---

**References:**
- [Parking Lot System](./12_PARKING_LOT_SYSTEM.md)
- [Design Patterns](./08_CREATIONAL_PATTERNS.md)


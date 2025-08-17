# Advanced Java Programming Glossary

A comprehensive reference guide for advanced Java programming terms and concepts with definitions, analogies, and sample code.

## A

  **Abstract classes**  
  Contain abstract methods that must be implemented by subclasses and concrete methods that can be used or overridden by subclasses.  
  *Analogy: Like a partially built house template - some rooms are complete and ready to use, while others are just outlined and need to be finished by the new owner.*
  ```java
  abstract class Animal {
      // Concrete method
      public void sleep() {
          System.out.println("Animal is sleeping");
      }
      
      // Abstract method - must be implemented by subclasses
      public abstract void makeSound();
  }
  
  class Dog extends Animal {
      @Override
      public void makeSound() {
          System.out.println("Woof!");
      }
  }
  ```
  
  **Abstract modifiers**  
  Used for classes and methods. Abstract classes cannot be instantiated and must be extended, while abstract methods must be implemented in subclasses.  
  *Analogy: Like a "Coming Soon" sign on a store - you know it will be a business, but it's not ready to serve customers yet.*
  ```java
  abstract class Vehicle {
      protected String brand;
      
      public Vehicle(String brand) {
          this.brand = brand;
      }
      
      // Abstract method
      public abstract void startEngine();
  }
  
  class Car extends Vehicle {
      public Car(String brand) {
          super(brand);
      }
      
      @Override
      public void startEngine() {
          System.out.println(brand + " car engine started");
      }
  }
  ```
  
  **Abstraction**  
  Simplifies complex systems by hiding unnecessary details and exposing only the essential features.  
  *Analogy: Like a car dashboard - you see the speedometer and fuel gauge (essential info) but not the engine's internal workings.*
  ```java
  // Interface provides abstraction
  interface PaymentProcessor {
      void processPayment(double amount);
  }
  
  class CreditCardProcessor implements PaymentProcessor {
      @Override
      public void processPayment(double amount) {
          // Complex internal logic hidden from user
          System.out.println("Processing $" + amount + " via credit card");
      }
  }
  
  // User only needs to know about processPayment method
  PaymentProcessor processor = new CreditCardProcessor();
  processor.processPayment(100.0);
  ```
  
  **Access modifiers**  
  Keywords that control the visibility and accessibility of classes, methods, and variables.  
  *Analogy: Like different security clearance levels in a building - some areas are public, some are employee-only, some are management-only.*
  ```java
  public class AccessExample {
      public String publicVar = "Everyone can see this";
      protected String protectedVar = "Package and subclasses can see this";
      String defaultVar = "Only package can see this";
      private String privateVar = "Only this class can see this";
      
      public void publicMethod() { }
      protected void protectedMethod() { }
      void defaultMethod() { }
      private void privateMethod() { }
  }
  ```
  
  **Anonymous inner class**  
  Does not have a name and is used to instantiate a class that may not be reused elsewhere. Anonymous inner classes are often used to create a subclass, implement interfaces, or extend classes with minimal code.  
  *Analogy: Like a disposable camera - built for one specific purpose and thrown away after use.*
  ```java
  // Anonymous inner class implementing Runnable
  Thread thread = new Thread(new Runnable() {
      @Override
      public void run() {
          System.out.println("Running in anonymous inner class");
      }
  });
  
  // Anonymous inner class extending a class
  JButton button = new JButton("Click me");
  button.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
          System.out.println("Button clicked!");
      }
  });
  ```
  
  **ArrayList**  
  Uses a dynamic array to store elements. ArrayList allows fast random access but accesses the elements more slowly when adding or removing elements from the middle.  
  *Analogy: Like a row of houses with street numbers - easy to find house #50, but difficult to squeeze a new house between #25 and #26.*
  ```java
  import java.util.ArrayList;
  
  ArrayList<String> fruits = new ArrayList<>();
  fruits.add("Apple");
  fruits.add("Banana");
  fruits.add("Orange");
  
  // Fast random access
  String secondFruit = fruits.get(1); // "Banana"
  
  // Slow insertion in middle
  fruits.add(1, "Mango"); // Shifts all elements after index 1
  
  System.out.println(fruits); // [Apple, Mango, Banana, Orange]
  ```

## B

  **Breadth-First Search (BFS)**  
  Fundamental algorithm in computer science used for traversing or searching tree or graph data structures. It explores nodes level by level, starting from the source node and moving outward.  
  *Analogy: Like exploring a building floor by floor - first check all rooms on the ground floor, then all rooms on the second floor, and so on.*
  ```java
  import java.util.*;
  
  class BFS {
      public void bfsTraversal(int[][] graph, int start) {
          Queue<Integer> queue = new LinkedList<>();
          boolean[] visited = new boolean[graph.length];
          
          queue.offer(start);
          visited[start] = true;
          
          while (!queue.isEmpty()) {
              int node = queue.poll();
              System.out.print(node + " ");
              
              for (int i = 0; i < graph[node].length; i++) {
                  if (graph[node][i] == 1 && !visited[i]) {
                      queue.offer(i);
                      visited[i] = true;
                  }
              }
          }
      }
  }
  ```
  
  **Buffer overflow**  
  Vulnerability that occurs when a program writes more data to a buffer than it can hold, potentially causing security issues.  
  *Analogy: Like trying to pour a gallon of water into a cup - it overflows and spills everywhere, potentially causing damage.*
  ```java
  // Example of preventing buffer overflow
  public class SafeBuffer {
      private char[] buffer = new char[10];
      private int position = 0;
      
      public boolean addChar(char c) {
          if (position < buffer.length) {
              buffer[position++] = c;
              return true;
          } else {
              System.out.println("Buffer overflow prevented!");
              return false;
          }
      }
  }
  ```
  
  **BufferedInputStream**  
  Class used to buffer input.  
  *Analogy: Like having a water reservoir that collects water in batches instead of drop by drop.*
  ```java
  import java.io.*;
  
  public class BufferedInputExample {
      public static void readFile(String filename) {
          try (BufferedInputStream bis = new BufferedInputStream(
                  new FileInputStream(filename))) {
              
              int data;
              while ((data = bis.read()) != -1) {
                  System.out.print((char) data);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  **BufferedOutputStream**  
  Class used to buffer output.  
  *Analogy: Like collecting mail in a mailbag before delivering it all at once, rather than making individual trips.*
  ```java
  import java.io.*;
  
  public class BufferedOutputExample {
      public static void writeFile(String filename, String content) {
          try (BufferedOutputStream bos = new BufferedOutputStream(
                  new FileOutputStream(filename))) {
              
              bos.write(content.getBytes());
              bos.flush(); // Force write buffered data
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  **BufferedReader**  
  Class in Java that reads text from a character-input stream, buffering characters to provide efficient reading of characters, arrays, and lines.  
  *Analogy: Like reading a newspaper section by section instead of letter by letter.*
  ```java
  import java.io.*;
  
  public class BufferedReaderExample {
      public static void readTextFile(String filename) {
          try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
              String line;
              while ((line = br.readLine()) != null) {
                  System.out.println(line);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  **BufferedWriter**  
  Class that improves the efficiency of writing data.  
  *Analogy: Like writing a letter on paper and sending it all at once rather than mailing each word separately.*
  ```java
  import java.io.*;
  
  public class BufferedWriterExample {
      public static void writeTextFile(String filename, String[] lines) {
          try (BufferedWriter bw = new BufferedWriter(new FileWriter(filename))) {
              for (String line : lines) {
                  bw.write(line);
                  bw.newLine();
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  **Byte stream**  
  Used to read and write data as raw bytes. It handles all types of data, such as text, images, and videos, making it versatile for various tasks.  
  *Analogy: Like a universal translator that can handle any type of information by breaking it down into basic units.*
  ```java
  import java.io.*;
  
  public class ByteStreamExample {
      public static void copyFile(String source, String destination) {
          try (FileInputStream fis = new FileInputStream(source);
               FileOutputStream fos = new FileOutputStream(destination)) {
              
              int byteData;
              while ((byteData = fis.read()) != -1) {
                  fos.write(byteData);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

## C

  **Character stream**  
  Deals with character data or text.  
  *Analogy: Like a translator specialized in text documents rather than binary files.*
  ```java
  import java.io.*;
  
  public class CharacterStreamExample {
      public static void copyTextFile(String source, String destination) {
          try (FileReader fr = new FileReader(source);
               FileWriter fw = new FileWriter(destination)) {
              
              int charData;
              while ((charData = fr.read()) != -1) {
                  fw.write(charData);
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```
  
  **Class**  
  Serves as a blueprint for creating objects.  
  *Analogy: Like an architectural blueprint that defines how to build a house.*
  ```java
  public class Car {
      private String brand;
      private String color;
      private int year;
      
      public Car(String brand, String color, int year) {
          this.brand = brand;
          this.color = color;
          this.year = year;
      }
      
      public void start() {
          System.out.println("Car is starting");
      }
      
      public void stop() {
          System.out.println("Car has stopped");
      }
  }
  
  // Creating objects from the class
  Car myCar = new Car("Toyota", "Red", 2023);
  myCar.start();
  ```
  
  **Class definition**  
  Indicates the purpose of the program.  
  *Analogy: Like the title and description of a book that tells you what it's about.*
  ```java
  /**
   * Class definition for a simple calculator
   * Purpose: Perform basic arithmetic operations
   */
  public class Calculator {
      public int add(int a, int b) {
          return a + b;
      }
      
      public int subtract(int a, int b) {
          return a - b;
      }
      
      public int multiply(int a, int b) {
          return a * b;
      }
      
      public double divide(int a, int b) {
          if (b != 0) {
              return (double) a / b;
          }
          throw new IllegalArgumentException("Cannot divide by zero");
      }
  }
  ```
  
  **close()**  
  Method that closes writer, saves data, and releases resources.  
  *Analogy: Like closing a file cabinet and locking it after you're done working.*
  ```java
  import java.io.*;
  
  public class CloseExample {
      public static void writeAndClose(String filename, String content) {
          FileWriter writer = null;
          try {
              writer = new FileWriter(filename);
              writer.write(content);
          } catch (IOException e) {
              e.printStackTrace();
          } finally {
              if (writer != null) {
                  try {
                      writer.close(); // Always close to release resources
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
              }
          }
      }
      
      // Better approach with try-with-resources
      public static void writeAndAutoClose(String filename, String content) {
          try (FileWriter writer = new FileWriter(filename)) {
              writer.write(content);
          } catch (IOException e) {
              e.printStackTrace();
          } // Automatically closes
      }
  }
  ```
  
  **Code reusability**  
  Allowing a class to inherit methods and properties from another class, eliminating the need to rewrite existing code.  
  *Analogy: Like using a template for multiple documents instead of starting from scratch each time.*
  ```java
  // Base class with reusable code
  class Animal {
      protected String name;
      
      public Animal(String name) {
          this.name = name;
      }
      
      public void eat() {
          System.out.println(name + " is eating");
      }
      
      public void sleep() {
          System.out.println(name + " is sleeping");
      }
  }
  
  // Reusing Animal's code in Dog class
  class Dog extends Animal {
      public Dog(String name) {
          super(name);
      }
      
      public void bark() {
          System.out.println(name + " is barking");
      }
  }
  
  // Reusing Animal's code in Cat class
  class Cat extends Animal {
      public Cat(String name) {
          super(name);
      }
      
      public void meow() {
          System.out.println(name + " is meowing");
      }
  }
  ```
  
  **Collection**  
  Group of objects in Java that store, retrieve, manipulate, and communicate aggregate data.  
  *Analogy: Like different types of containers for organizing items - boxes, bags, shelves, etc.*
  ```java
  import java.util.*;
  
  public class CollectionExample {
      public static void main(String[] args) {
          // List - ordered, allows duplicates
          List<String> names = new ArrayList<>();
          names.add("Alice");
          names.add("Bob");
          names.add("Alice"); // Duplicate allowed
          
          // Set - no duplicates
          Set<String> uniqueNames = new HashSet<>();
          uniqueNames.add("Alice");
          uniqueNames.add("Bob");
          uniqueNames.add("Alice"); // Duplicate ignored
          
          // Map - key-value pairs
          Map<String, Integer> ages = new HashMap<>();
          ages.put("Alice", 25);
          ages.put("Bob", 30);
          
          System.out.println("Names: " + names);
          System.out.println("Unique names: " + uniqueNames);
          System.out.println("Ages: " + ages);
      }
  }
  ```
  
  **Compile-time polymorphism**  
  Occurs when multiple methods belonging to the same class have the same name but differ in the number or type of parameters. It is also known as method overloading.  
  *Analogy: Like having multiple versions of a "print" function - one for photos, one for documents, one for labels.*
  ```java
  public class MathOperations {
      // Method overloading - same name, different parameters
      public int add(int a, int b) {
          return a + b;
      }
      
      public double add(double a, double b) {
          return a + b;
      }
      
      public int add(int a, int b, int c) {
          return a + b + c;
      }
      
      public String add(String a, String b) {
          return a + b;
      }
  }
  
  // Usage
  MathOperations math = new MathOperations();
  System.out.println(math.add(5, 3));           // Calls int version
  System.out.println(math.add(5.5, 3.2));      // Calls double version
  System.out.println(math.add(1, 2, 3));       // Calls three-parameter version
  System.out.println(math.add("Hello", "World")); // Calls String version
  ```
  
  **Constructor**  
  Initializes objects by assigning values to their attributes when it is created.  
  *Analogy: Like the setup process when you first unbox a new phone - it configures the initial settings.*
  ```java
  public class Student {
      private String name;
      private int age;
      private String major;
      
      // Default constructor
      public Student() {
          this.name = "Unknown";
          this.age = 0;
          this.major = "Undeclared";
      }
      
      // Parameterized constructor
      public Student(String name, int age, String major) {
          this.name = name;
          this.age = age;
          this.major = major;
      }
      
      // Constructor overloading
      public Student(String name, int age) {
          this.name = name;
          this.age = age;
          this.major = "Undeclared";
      }
      
      public void displayInfo() {
          System.out.println("Name: " + name + ", Age: " + age + ", Major: " + major);
      }
  }
  
  // Usage
  Student student1 = new Student(); // Default constructor
  Student student2 = new Student("Alice", 20, "Computer Science");
  Student student3 = new Student("Bob", 19);
  ```
  
  **containsKey()**  
  Method that checks if a key exists.  
  *Analogy: Like checking if a specific key is on your keyring before trying to use it.*
  ```java
  import java.util.HashMap;
  
  public class ContainsKeyExample {
      public static void main(String[] args) {
          HashMap<String, Integer> scores = new HashMap<>();
          scores.put("Alice", 95);
          scores.put("Bob", 87);
          scores.put("Carol", 92);
          
          // Check if key exists before accessing
          if (scores.containsKey("Alice")) {
              System.out.println("Alice's score: " + scores.get("Alice"));
          } else {
              System.out.println("Alice's score not found");
          }
          
          // Safe way to get value with default
          String student = "David";
          int score = scores.containsKey(student) ? scores.get(student) : 0;
          System.out.println(student + "'s score: " + score);
      }
  }
  ```
  
  **create directory**  
  Operation that allows users to create a new directory under the "Documents" folder. This functionality helps organize files by categorizing them into separate folders, making it easier to manage and access them later.  
  *Analogy: Like creating a new folder in a filing cabinet to organize your documents.*
  ```java
  import java.io.File;
  import java.nio.file.*;
  
  public class CreateDirectoryExample {
      public static void createDirectory(String dirPath) {
          // Method 1: Using File class
          File directory = new File(dirPath);
          if (!directory.exists()) {
              if (directory.mkdirs()) {
                  System.out.println("Directory created: " + dirPath);
              } else {
                  System.out.println("Failed to create directory");
              }
          } else {
              System.out.println("Directory already exists");
          }
      }
      
      public static void createDirectoryNIO(String dirPath) {
          // Method 2: Using NIO (recommended)
          try {
              Path path = Paths.get(dirPath);
              if (!Files.exists(path)) {
                  Files.createDirectories(path);
                  System.out.println("Directory created: " + dirPath);
              } else {
                  System.out.println("Directory already exists");
              }
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```

## D

  **Data hiding**  
  Hides private data, preventing unauthorized access and maintaining security.  
  *Analogy: Like keeping your personal diary locked in a drawer - others can ask you about your day, but they can't read it directly.*
  ```java
  public class BankAccount {
      private double balance; // Hidden from external access
      private String accountNumber;
      
      public BankAccount(String accountNumber, double initialBalance) {
          this.accountNumber = accountNumber;
          this.balance = initialBalance;
      }
      
      // Controlled access through public methods
      public double getBalance() {
          return balance;
      }
      
      public void deposit(double amount) {
          if (amount > 0) {
              balance += amount;
              System.out.println("Deposited: $" + amount);
          }
      }
      
      public boolean withdraw(double amount) {
          if (amount > 0 && amount <= balance) {
              balance -= amount;
              System.out.println("Withdrawn: $" + amount);
              return true;
          }
          System.out.println("Invalid withdrawal amount");
          return false;
      }
      
      // balance cannot be directly accessed or modified from outside
  }
  ```
  
  **database storage**  
  A method of storing dates in a specific format required by databases.  
  *Analogy: Like formatting addresses in a standard way so the postal system can process them correctly.*
  ```java
  import java.sql.*;
  import java.time.LocalDate;
  
  public class DatabaseStorageExample {
      public void storeDateInDatabase(LocalDate date) {
          String url = "jdbc:mysql://localhost:3306/mydb";
          String sql = "INSERT INTO events (event_date) VALUES (?)";
          
          try (Connection conn = DriverManager.getConnection(url);
               PreparedStatement stmt = conn.prepareStatement(sql)) {
              
              // Convert LocalDate to SQL Date for database storage
              stmt.setDate(1, Date.valueOf(date));
              stmt.executeUpdate();
              
              System.out.println("Date stored in database: " + date);
          } catch (SQLException e) {
              e.printStackTrace();
          }
      }
      
      public LocalDate retrieveDateFromDatabase(int eventId) {
          String url = "jdbc:mysql://localhost:3306/mydb";
          String sql = "SELECT event_date FROM events WHERE id = ?";
          
          try (Connection conn = DriverManager.getConnection(url);
               PreparedStatement stmt = conn.prepareStatement(sql)) {
              
              stmt.setInt(1, eventId);
              ResultSet rs = stmt.executeQuery();
              
              if (rs.next()) {
                  Date sqlDate = rs.getDate("event_date");
                  return sqlDate.toLocalDate(); // Convert back to LocalDate
              }
          } catch (SQLException e) {
              e.printStackTrace();
          }
          return null;
      }
  }
  ```
  
  **date format**  
  A pattern used to represent dates as strings, such as dd/MM/yyyy or yyyy-MM-dd.  
  *Analogy: Like different ways to write the same address - some prefer "Street, City, State" while others prefer "City, State, Street".*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  
  public class DateFormatExample {
      public static void main(String[] args) {
          LocalDate date = LocalDate.of(2023, 12, 25);
          
          // Different date formats
          DateTimeFormatter format1 = DateTimeFormatter.ofPattern("dd/MM/yyyy");
          DateTimeFormatter format2 = DateTimeFormatter.ofPattern("yyyy-MM-dd");
          DateTimeFormatter format3 = DateTimeFormatter.ofPattern("MMM dd, yyyy");
          DateTimeFormatter format4 = DateTimeFormatter.ofPattern("EEEE, MMMM dd, yyyy");
          
          System.out.println("Format 1: " + date.format(format1)); // 25/12/2023
          System.out.println("Format 2: " + date.format(format2)); // 2023-12-25
          System.out.println("Format 3: " + date.format(format3)); // Dec 25, 2023
          System.out.println("Format 4: " + date.format(format4)); // Monday, December 25, 2023
      }
  }
  ```
  
  **date manipulation**  
  The process of changing dates by adding or subtracting days, months, or years.  
  *Analogy: Like using a calendar to figure out what date it will be in 30 days or what date it was 2 weeks ago.*
  ```java
  import java.time.LocalDate;
  
  public class DateManipulationExample {
      public static void main(String[] args) {
          LocalDate today = LocalDate.now();
          System.out.println("Today: " + today);
          
          // Adding time
          LocalDate nextWeek = today.plusWeeks(1);
          LocalDate nextMonth = today.plusMonths(1);
          LocalDate nextYear = today.plusYears(1);
          
          System.out.println("Next week: " + nextWeek);
          System.out.println("Next month: " + nextMonth);
          System.out.println("Next year: " + nextYear);
          
          // Subtracting time
          LocalDate lastWeek = today.minusWeeks(1);
          LocalDate lastMonth = today.minusMonths(1);
          LocalDate lastYear = today.minusYears(1);
          
          System.out.println("Last week: " + lastWeek);
          System.out.println("Last month: " + lastMonth);
          System.out.println("Last year: " + lastYear);
          
          // Chaining operations
          LocalDate complexDate = today.plusMonths(2).minusDays(5).plusYears(1);
          System.out.println("Complex manipulation: " + complexDate);
      }
  }
  ```
  
  **date validation**  
  The process of checking whether a date is valid according to a specific format.  
  *Analogy: Like checking if someone wrote a valid phone number - making sure it has the right number of digits and format.*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  import java.time.format.DateTimeParseException;
  
  public class DateValidationExample {
      public static boolean isValidDate(String dateString, String pattern) {
          try {
              DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
              LocalDate.parse(dateString, formatter);
              return true;
          } catch (DateTimeParseException e) {
              return false;
          }
      }
      
      public static boolean isDateInRange(LocalDate date, LocalDate start, LocalDate end) {
          return !date.isBefore(start) && !date.isAfter(end);
      }
      
      public static void main(String[] args) {
          // Test date validation
          System.out.println(isValidDate("25/12/2023", "dd/MM/yyyy")); // true
          System.out.println(isValidDate("32/12/2023", "dd/MM/yyyy")); // false - invalid day
          System.out.println(isValidDate("25/13/2023", "dd/MM/yyyy")); // false - invalid month
          
          // Test date range validation
          LocalDate testDate = LocalDate.of(2023, 6, 15);
          LocalDate startDate = LocalDate.of(2023, 1, 1);
          LocalDate endDate = LocalDate.of(2023, 12, 31);
          
          System.out.println("Date in range: " + isDateInRange(testDate, startDate, endDate));
      }
  }
  ```
  
  **DateTimeFormatter**  
  A Java class used to define patterns for parsing and formatting dates and times.  
  *Analogy: Like a template that tells you exactly how to write dates - whether to put the day first, use slashes or dashes, spell out months, etc.*
  ```java
  import java.time.LocalDateTime;
  import java.time.format.DateTimeFormatter;
  
  public class DateTimeFormatterExample {
      public static void main(String[] args) {
          LocalDateTime now = LocalDateTime.now();
          
          // Predefined formatters
          System.out.println("ISO_LOCAL_DATE: " + now.format(DateTimeFormatter.ISO_LOCAL_DATE));
          System.out.println("ISO_LOCAL_TIME: " + now.format(DateTimeFormatter.ISO_LOCAL_TIME));
          System.out.println("ISO_LOCAL_DATE_TIME: " + now.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
          
          // Custom formatters
          DateTimeFormatter custom1 = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");
          DateTimeFormatter custom2 = DateTimeFormatter.ofPattern("MMM dd, yyyy 'at' h:mm a");
          DateTimeFormatter custom3 = DateTimeFormatter.ofPattern("EEEE, MMMM dd, yyyy");
          
          System.out.println("Custom 1: " + now.format(custom1));
          System.out.println("Custom 2: " + now.format(custom2));
          System.out.println("Custom 3: " + now.format(custom3));
          
          // Parsing dates with formatter
          String dateString = "25-12-2023 14:30:45";
          LocalDateTime parsed = LocalDateTime.parse(dateString, custom1);
          System.out.println("Parsed date: " + parsed);
      }
  }
  ```
  
  **DateTimeParseException**  
  An exception that is thrown when a date or time string cannot be parsed into a valid object, specific to Java's java.time.format package.  
  *Analogy: Like getting an error message when you try to enter an invalid phone number in a form.*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  import java.time.format.DateTimeParseException;
  
  public class DateTimeParseExceptionExample {
      public static LocalDate parseDate(String dateString, String pattern) {
          try {
              DateTimeFormatter formatter = DateTimeFormatter.ofPattern(pattern);
              return LocalDate.parse(dateString, formatter);
          } catch (DateTimeParseException e) {
              System.out.println("Parse error: " + e.getMessage());
              System.out.println("Invalid date string: " + dateString);
              System.out.println("Expected pattern: " + pattern);
              return null;
          }
      }
      
      public static void main(String[] args) {
          // Valid parsing
          LocalDate validDate = parseDate("25/12/2023", "dd/MM/yyyy");
          System.out.println("Valid date: " + validDate);
          
          // Invalid parsing examples
          parseDate("32/12/2023", "dd/MM/yyyy"); // Invalid day
          parseDate("25/13/2023", "dd/MM/yyyy"); // Invalid month
          parseDate("25-12-2023", "dd/MM/yyyy"); // Wrong separator
          parseDate("2023/12/25", "dd/MM/yyyy"); // Wrong order
      }
  }
  ```
  
  **Daylight saving time (DST)**
  A practice of moving clocks forward during warmer months to extend evening daylight.
  *Analogy: Like agreeing as a community to pretend time jumped forward an hour to get more sunlight in the evening*
  ```java
  import java.time.ZonedDateTime;
  import java.time.ZoneId;
  
  public class DaylightSavingExample {
      public void demonstrateDST() {
          ZoneId newYork = ZoneId.of("America/New_York");
          
          // Before DST transition
          ZonedDateTime beforeDST = ZonedDateTime.of(2024, 3, 10, 1, 30, 0, 0, newYork);
          
          // Add 2 hours - will skip the "spring forward" hour
          ZonedDateTime afterDST = beforeDST.plusHours(2);
          
          System.out.println("Before DST: " + beforeDST);
          System.out.println("After DST: " + afterDST);
          System.out.println("DST Active: " + afterDST.getZone().getRules().isDaylightSavings(afterDST.toInstant()));
      }
  }
  ```
  
  **Declaration**
  Defines a reference variable.
  *Analogy: Like putting a name tag on an empty box - you've declared what the box will hold, but haven't put anything in it yet*
  ```java
  public class DeclarationExample {
      public void demonstrateDeclarations() {
          // Variable declarations
          int number;           // Declared but not initialized
          String message;       // Declared but not initialized
          
          // Array declaration
          int[] numbers;        // Declared but not initialized
          
          // Object declaration
          StringBuilder builder; // Declared but not initialized
          
          // Later initialization
          number = 42;
          message = "Hello World";
          numbers = new int[5];
          builder = new StringBuilder();
      }
  }
  ```
  
  **Decoupling**
  Separation of code that uses an interface from the code that implements it, resulting in loosely coupled systems that are easier to maintain.
  *Analogy: Like using a universal remote control - you don't need to know how each TV brand works internally, just press the buttons*
  ```java
  // Interface defines the contract
  interface PaymentProcessor {
      boolean processPayment(double amount);
  }
  
  // Different implementations
  class CreditCardProcessor implements PaymentProcessor {
      public boolean processPayment(double amount) {
          System.out.println("Processing $" + amount + " via credit card");
          return true;
      }
  }
  
  class PayPalProcessor implements PaymentProcessor {
      public boolean processPayment(double amount) {
          System.out.println("Processing $" + amount + " via PayPal");
          return true;
      }
  }
  
  // Client code is decoupled from specific implementations
  class ShoppingCart {
      private PaymentProcessor processor;
      
      public ShoppingCart(PaymentProcessor processor) {
          this.processor = processor; // Depends on interface, not implementation
      }
      
      public void checkout(double total) {
          processor.processPayment(total); // Doesn't care which implementation
      }
  }
  ```
  
  **Default constructor**
  Automatically provided by Java when no constructors are explicitly defined in a class. It takes no parameters and is used to initialize the object with default values.
  *Analogy: Like a basic assembly line that creates a standard product when you don't specify any custom options*
  ```java
  public class DefaultConstructorExample {
      private String name;
      private int age;
      
      // No explicit constructor defined, so Java provides:
      // public DefaultConstructorExample() { }
      
      public static void main(String[] args) {
          DefaultConstructorExample obj = new DefaultConstructorExample();
          System.out.println("Name: " + obj.name); // null
          System.out.println("Age: " + obj.age);   // 0
      }
  }
  
  class WithExplicitConstructor {
      private String name;
      
      // Once we define ANY constructor, default constructor is NOT provided
      public WithExplicitConstructor(String name) {
          this.name = name;
      }
      
      // This would cause compilation error:
      // WithExplicitConstructor obj = new WithExplicitConstructor(); // Error!
  }
  ```
  
  **Default modifier**
  Used when no access modifier is specified. It allows an element to be accessible only within the same package.
  *Analogy: Like having a shared folder that everyone in your office building can access, but not people from other buildings*
  ```java
  package com.example.packageA;
  
  public class DefaultModifierExample {
      String packagePrivateField = "Accessible within package"; // default access
      int packagePrivateMethod() { return 42; }                 // default access
  }
  
  class AnotherClassInSamePackage {
      public void accessDefaultMembers() {
          DefaultModifierExample obj = new DefaultModifierExample();
          System.out.println(obj.packagePrivateField); // OK - same package
          System.out.println(obj.packagePrivateMethod()); // OK - same package
      }
  }
  
  // In a different package:
  package com.example.packageB;
  import com.example.packageA.DefaultModifierExample;
  
  class ClassInDifferentPackage {
      public void tryAccess() {
          DefaultModifierExample obj = new DefaultModifierExample();
          // System.out.println(obj.packagePrivateField); // Compilation error!
          // obj.packagePrivateMethod();                   // Compilation error!
      }
  }
  ```
  
  **Defining format**
  The process of creating a DateTimeFormatter with a desired pattern.
  *Analogy: Like creating a template for how you want your dates to look - deciding the style before you start writing*
  ```java
  import java.time.LocalDateTime;
  import java.time.format.DateTimeFormatter;
  
  public class DefiningFormatExample {
      public void defineCustomFormats() {
          LocalDateTime dateTime = LocalDateTime.now();
          
          // Define various formats
          DateTimeFormatter shortFormat = DateTimeFormatter.ofPattern("dd/MM/yy");
          DateTimeFormatter longFormat = DateTimeFormatter.ofPattern("EEEE, MMMM dd, yyyy 'at' HH:mm");
          DateTimeFormatter timeOnlyFormat = DateTimeFormatter.ofPattern("HH:mm:ss");
          DateTimeFormatter customFormat = DateTimeFormatter.ofPattern("'Today is' dd-MMM-yyyy");
          
          System.out.println("Short: " + dateTime.format(shortFormat));
          System.out.println("Long: " + dateTime.format(longFormat));
          System.out.println("Time only: " + dateTime.format(timeOnlyFormat));
          System.out.println("Custom: " + dateTime.format(customFormat));
      }
  }
  ```
  
  **Delete directory**
  Operation that allows users to remove a directory if it exists. However, the directory must be empty before it can be deleted.
  *Analogy: Like trying to throw away a box - you need to empty it first or the garbage collector won't take it*
  ```java
  import java.io.File;
  import java.io.IOException;
  import java.nio.file.Files;
  import java.nio.file.Path;
  import java.nio.file.Paths;
  
  public class DeleteDirectoryExample {
      public void deleteEmptyDirectory() {
          File dir = new File("emptyDirectory");
          
          if (dir.exists() && dir.isDirectory()) {
              if (dir.delete()) {
                  System.out.println("Directory deleted successfully");
              } else {
                  System.out.println("Failed to delete directory (may not be empty)");
              }
          }
      }
      
      public void deleteDirectoryWithFiles() throws IOException {
          Path dirPath = Paths.get("directoryWithFiles");
          
          if (Files.exists(dirPath)) {
              // Delete all files in directory first
              Files.walk(dirPath)
                  .sorted((a, b) -> b.compareTo(a)) // Reverse order to delete files before directories
                  .forEach(path -> {
                      try {
                          Files.delete(path);
                      } catch (IOException e) {
                          System.err.println("Failed to delete: " + path);
                      }
                  });
          }
      }
  }
  ```
  
  **delete()**
  Returns true if the deletion is successful. The developer is notified if the directory contains files or does not exist.
  *Analogy: Like a confirmation receipt - it tells you whether your deletion request was successful or failed*
  ```java
  import java.io.File;
  
  public class DeleteMethodExample {
      public void demonstrateDelete() {
          File file = new File("testFile.txt");
          File directory = new File("testDirectory");
          File nonExistent = new File("doesNotExist.txt");
          
          // Try to delete file
          if (file.exists()) {
              boolean fileDeleted = file.delete();
              System.out.println("File deletion successful: " + fileDeleted);
          }
          
          // Try to delete empty directory
          if (directory.exists() && directory.isDirectory()) {
              boolean dirDeleted = directory.delete();
              System.out.println("Directory deletion successful: " + dirDeleted);
          }
          
          // Try to delete non-existent file
          boolean nonExistentDeleted = nonExistent.delete();
          System.out.println("Non-existent file deletion: " + nonExistentDeleted); // false
      }
  }
  ```
  
  **Delimiters**
  Characters or sequences used to separate strings into parts, such as commas or spaces.
  *Analogy: Like using commas in a grocery list - they separate each item so you know where one ends and the next begins*
  ```java
  import java.util.StringTokenizer;
  
  public class DelimitersExample {
      public void demonstrateDelimiters() {
          String csvData = "apple,banana,orange,grape";
          String spaceData = "Java is a programming language";
          String multiDelimiter = "apple;banana,orange:grape";
          
          // Using split() with comma delimiter
          String[] fruits = csvData.split(",");
          System.out.println("Fruits: ");
          for (String fruit : fruits) {
              System.out.println("  " + fruit.trim());
          }
          
          // Using split() with space delimiter
          String[] words = spaceData.split(" ");
          System.out.println("\nWords: ");
          for (String word : words) {
              System.out.println("  " + word);
          }
          
          // Using multiple delimiters
          String[] items = multiDelimiter.split("[;,:]+");
          System.out.println("\nMulti-delimiter items: ");
          for (String item : items) {
              System.out.println("  " + item);
          }
      }
  }
  ```
  
  **Dequeue**
  Removes an item from the front of the queue.
  *Analogy: Like being the first person in line at a coffee shop - you get served and leave, making room for the next person*
  ```java
  import java.util.LinkedList;
  import java.util.Queue;
  import java.util.ArrayDeque;
  import java.util.Deque;
  
  public class DequeueExample {
      public void demonstrateDequeue() {
          // Basic queue dequeue (remove from front)
          Queue<String> queue = new LinkedList<>();
          queue.offer("First");
          queue.offer("Second");
          queue.offer("Third");
          
          System.out.println("Queue before dequeue: " + queue);
          String removed = queue.poll(); // Dequeue operation
          System.out.println("Removed: " + removed);
          System.out.println("Queue after dequeue: " + queue);
          
          // Double-ended queue (Deque) - can remove from both ends
          Deque<Integer> deque = new ArrayDeque<>();
          deque.addFirst(1);
          deque.addLast(2);
          deque.addLast(3);
          
          System.out.println("\nDeque: " + deque);
          int fromFront = deque.removeFirst(); // Remove from front
          int fromRear = deque.removeLast();   // Remove from rear
          
          System.out.println("Removed from front: " + fromFront);
          System.out.println("Removed from rear: " + fromRear);
          System.out.println("Deque after removals: " + deque);
      }
  }
  ```
  
  **Directories**
  Folders that store files and other folders inside them. They help organize data in a clear and easy-to-use structure.
  *Analogy: Like a filing cabinet with labeled drawers and folders inside - everything has its place and you can find what you need*
  ```java
  import java.io.File;
  import java.io.IOException;
  import java.nio.file.Files;
  import java.nio.file.Path;
  import java.nio.file.Paths;
  
  public class DirectoriesExample {
      public void workWithDirectories() throws IOException {
          // Create directory structure
          File mainDir = new File("documents");
          File subDir1 = new File(mainDir, "photos");
          File subDir2 = new File(mainDir, "videos");
          
          // Create directories
          if (mainDir.mkdirs()) {
              System.out.println("Created directory: " + mainDir.getPath());
          }
          subDir1.mkdir();
          subDir2.mkdir();
          
          // List directory contents
          System.out.println("Contents of " + mainDir.getName() + ":");
          File[] contents = mainDir.listFiles();
          if (contents != null) {
              for (File item : contents) {
                  System.out.println("  " + (item.isDirectory() ? "[DIR] " : "[FILE] ") + item.getName());
              }
          }
          
          // Using modern Path API
          Path documentsPath = Paths.get("documents");
          System.out.println("\nDirectory exists: " + Files.exists(documentsPath));
          System.out.println("Is directory: " + Files.isDirectory(documentsPath));
          System.out.println("Absolute path: " + documentsPath.toAbsolutePath());
      }
  }
  ```
  
  **Duration**
  A class representing a time-based amount of time, such as hours or minutes.
  *Analogy: Like a stopwatch that measures how much time passed between two moments*
  ```java
  import java.time.Duration;
  import java.time.LocalTime;
  import java.time.temporal.ChronoUnit;
  
  public class DurationExample {
      public void demonstrateDuration() {
          // Create durations in different ways
          Duration oneHour = Duration.ofHours(1);
          Duration thirtyMinutes = Duration.ofMinutes(30);
          Duration twoAndHalfHours = Duration.ofMinutes(150);
          
          System.out.println("1 hour: " + oneHour);
          System.out.println("30 minutes: " + thirtyMinutes);
          System.out.println("2.5 hours: " + twoAndHalfHours);
          
          // Calculate duration between times
          LocalTime start = LocalTime.of(9, 0);    // 9:00 AM
          LocalTime end = LocalTime.of(17, 30);    // 5:30 PM
          Duration workDay = Duration.between(start, end);
          
          System.out.println("\nWork day duration: " + workDay);
          System.out.println("Work day in hours: " + workDay.toHours());
          System.out.println("Work day in minutes: " + workDay.toMinutes());
          
          // Duration operations
          Duration breakTime = Duration.ofMinutes(60);
          Duration actualWorkTime = workDay.minus(breakTime);
          
          System.out.println("Actual work time: " + actualWorkTime);
          System.out.println("Is work day longer than 8 hours? " + workDay.compareTo(Duration.ofHours(8)) > 0);
      }
  }
  ```
  
  **Dynamic method resolution**
  Enables changing behavior at runtime and supports complex systems.
  *Analogy: Like a smart GPS that chooses the best route while you're driving, adapting to current traffic conditions*
  ```java
  abstract class Animal {
      abstract void makeSound();
      
      public void introduce() {
          System.out.print("I am an animal and I ");
          makeSound(); // This will be resolved at runtime
      }
  }
  
  class Dog extends Animal {
      @Override
      void makeSound() {
          System.out.println("bark!");
      }
  }
  
  class Cat extends Animal {
      @Override
      void makeSound() {
          System.out.println("meow!");
      }
  }
  
  class Bird extends Animal {
      @Override
      void makeSound() {
          System.out.println("chirp!");
      }
  }
  
  public class DynamicMethodResolutionExample {
      public void demonstrateDynamicResolution() {
          // The actual method called is determined at runtime
          Animal[] animals = {
              new Dog(),
              new Cat(), 
              new Bird()
          };
          
          for (Animal animal : animals) {
              // Method resolution happens at runtime based on actual object type
              animal.introduce(); // Different behavior for each animal type
          }
          
          // Runtime method resolution with user input simulation
          Animal pet = getRandomAnimal();
          pet.makeSound(); // We don't know which method will be called until runtime
      }
      
      private Animal getRandomAnimal() {
          int random = (int) (Math.random() * 3);
          switch (random) {
              case 0: return new Dog();
              case 1: return new Cat();
              default: return new Bird();
          }
      }
  }
  ```

## E

  **Encapsulation**
  Bundling an object's data and the methods that operate on that data into a single unit, typically a class.
  *Analogy: Like a medicine capsule - all the active ingredients are safely contained inside, and you can only access them through the proper opening*
  ```java
  public class BankAccount {
      // Private data - encapsulated/hidden from outside
      private double balance;
      private String accountNumber;
      private String ownerName;
      
      // Constructor
      public BankAccount(String accountNumber, String ownerName, double initialBalance) {
          this.accountNumber = accountNumber;
          this.ownerName = ownerName;
          this.balance = initialBalance;
      }
      
      // Public methods - controlled access to data
      public void deposit(double amount) {
          if (amount > 0) {
              balance += amount;
              System.out.println("Deposited: $" + amount);
          }
      }
      
      public boolean withdraw(double amount) {
          if (amount > 0 && amount <= balance) {
              balance -= amount;
              System.out.println("Withdrawn: $" + amount);
              return true;
          }
          return false;
      }
      
      // Getter method - controlled read access
      public double getBalance() {
          return balance;
      }
      
      public String getAccountInfo() {
          return "Account: " + accountNumber + ", Owner: " + ownerName;
      }
      
      // No direct setter for balance - can only be modified through deposit/withdraw
  }
  ```
  **Enqueue**
  Adds an item to the back of the queue.
  *Analogy: Like joining the end of a line at a store - you wait your turn behind everyone who was there before you*
  ```java
  import java.util.LinkedList;
  import java.util.Queue;
  import java.util.ArrayDeque;
  
  public class EnqueueExample {
      public void demonstrateEnqueue() {
          Queue<String> customerQueue = new LinkedList<>();
          
          // Enqueue operations - adding to the back of queue
          customerQueue.offer("Alice");   // First customer
          customerQueue.offer("Bob");     // Second customer  
          customerQueue.offer("Charlie"); // Third customer
          
          System.out.println("Queue after enqueuing: " + customerQueue);
          // Output: [Alice, Bob, Charlie]
          
          // Process customers (dequeue from front)
          while (!customerQueue.isEmpty()) {
              String customer = customerQueue.poll();
              System.out.println("Now serving: " + customer);
          }
      }
      
      public void simulateTicketLine() {
          Queue<String> ticketLine = new ArrayDeque<>();
          
          // People joining the line (enqueue)
          System.out.println("People joining ticket line:");
          enqueueCustomer(ticketLine, "John");
          enqueueCustomer(ticketLine, "Mary");
          enqueueCustomer(ticketLine, "David");
          
          System.out.println("Final queue: " + ticketLine);
      }
      
      private void enqueueCustomer(Queue<String> queue, String customer) {
          queue.offer(customer);
          System.out.println(customer + " joined the line. Queue: " + queue);
      }
  }
  ```
  **Epoch**
  The starting point of time used in computing, typically January 1, 1970.
  *Analogy: Like having a universal birthday for all computers - they all start counting time from the same moment*
  ```java
  import java.time.Instant;
  import java.time.LocalDateTime;
  import java.time.ZoneId;
  import java.time.format.DateTimeFormatter;
  
  public class EpochExample {
      public void demonstrateEpoch() {
          // The Unix epoch: January 1, 1970, 00:00:00 UTC
          Instant epoch = Instant.EPOCH;
          System.out.println("Unix Epoch: " + epoch);
          
          // Convert epoch to readable format
          LocalDateTime epochDateTime = LocalDateTime.ofInstant(epoch, ZoneId.of("UTC"));
          System.out.println("Epoch in UTC: " + epochDateTime);
          
          // Current time since epoch
          Instant now = Instant.now();
          System.out.println("Current instant: " + now);
          
          // Seconds since epoch
          long secondsSinceEpoch = now.getEpochSecond();
          System.out.println("Seconds since epoch: " + secondsSinceEpoch);
          
          // Milliseconds since epoch
          long millisSinceEpoch = now.toEpochMilli();
          System.out.println("Milliseconds since epoch: " + millisSinceEpoch);
          
          // Create time from epoch seconds
          Instant specificTime = Instant.ofEpochSecond(1609459200L); // Jan 1, 2021
          LocalDateTime specificDateTime = LocalDateTime.ofInstant(specificTime, ZoneId.systemDefault());
          System.out.println("Specific time from epoch: " + specificDateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
      }
  }
  ```
  **Epoch time**
  The number of seconds or milliseconds since January 1, 1970 (Unix epoch).
  *Analogy: Like an odometer that never resets - it keeps counting the total time that has passed since computers started keeping track*
  ```java
  import java.time.Instant;
  import java.time.LocalDateTime;
  import java.time.ZoneId;
  
  public class EpochTimeExample {
      public void workWithEpochTime() {
          // Get current epoch time
          long currentEpochSeconds = Instant.now().getEpochSecond();
          long currentEpochMillis = Instant.now().toEpochMilli();
          
          System.out.println("Current epoch time (seconds): " + currentEpochSeconds);
          System.out.println("Current epoch time (milliseconds): " + currentEpochMillis);
          
          // Convert epoch time back to readable date
          Instant instantFromSeconds = Instant.ofEpochSecond(currentEpochSeconds);
          Instant instantFromMillis = Instant.ofEpochMilli(currentEpochMillis);
          
          LocalDateTime dateTimeFromSeconds = LocalDateTime.ofInstant(instantFromSeconds, ZoneId.systemDefault());
          LocalDateTime dateTimeFromMillis = LocalDateTime.ofInstant(instantFromMillis, ZoneId.systemDefault());
          
          System.out.println("From seconds: " + dateTimeFromSeconds);
          System.out.println("From milliseconds: " + dateTimeFromMillis);
          
          // Historical epoch times
          long birthdayOfInternet = 78796800L; // January 1, 1972 (approximately)
          Instant internetBirthday = Instant.ofEpochSecond(birthdayOfInternet);
          System.out.println("Early internet era: " + LocalDateTime.ofInstant(internetBirthday, ZoneId.of("UTC")));
          
          // Time calculations with epoch
          long oneYearInSeconds = 365 * 24 * 60 * 60L;
          long nextYearEpoch = currentEpochSeconds + oneYearInSeconds;
          Instant nextYear = Instant.ofEpochSecond(nextYearEpoch);
          System.out.println("One year from now: " + LocalDateTime.ofInstant(nextYear, ZoneId.systemDefault()));
      }
  }
  ```
  **Error message**
  A notification displayed when an operation, such as parsing, fails.
  *Analogy: Like a helpful librarian telling you exactly what went wrong when you can't find a book*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  import java.time.format.DateTimeParseException;
  
  public class ErrorMessageExample {
      public void demonstrateErrorMessages() {
          // Example 1: Date parsing error
          String invalidDate = "2024-13-45"; // Invalid month and day
          DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
          
          try {
              LocalDate date = LocalDate.parse(invalidDate, formatter);
          } catch (DateTimeParseException e) {
              System.out.println("Error Message: " + e.getMessage());
              System.out.println("Parsed Text: " + e.getParsedString());
              System.out.println("Error Index: " + e.getErrorIndex());
          }
          
          // Example 2: Number format error
          try {
              int number = Integer.parseInt("abc123");
          } catch (NumberFormatException e) {
              System.out.println("\nNumber Format Error: " + e.getMessage());
          }
          
          // Example 3: Array index error
          try {
              int[] array = {1, 2, 3};
              int value = array[5];
          } catch (ArrayIndexOutOfBoundsException e) {
              System.out.println("\nArray Error: " + e.getMessage());
          }
          
          // Example 4: Custom error messages
          validateAge(-5);
          validateEmail("invalid-email");
      }
      
      private void validateAge(int age) {
          try {
              if (age < 0) {
                  throw new IllegalArgumentException("Age cannot be negative. Provided: " + age);
              }
              if (age > 150) {
                  throw new IllegalArgumentException("Age seems unrealistic. Provided: " + age);
              }
              System.out.println("Valid age: " + age);
          } catch (IllegalArgumentException e) {
              System.out.println("Age Validation Error: " + e.getMessage());
          }
      }
      
      private void validateEmail(String email) {
          try {
              if (!email.contains("@")) {
                  throw new IllegalArgumentException("Email must contain @ symbol. Provided: '" + email + "'");
              }
              System.out.println("Valid email: " + email);
          } catch (IllegalArgumentException e) {
              System.out.println("Email Validation Error: " + e.getMessage());
          }
      }
  }
  ```
  **Extends keyword**
  Used to define that a class is inheriting from a superclass, enabling code reuse and extension.
  *Analogy: Like a child inheriting traits from their parent - they get all the family characteristics and can add their own unique features*
  ```java
  // Base class (superclass)
  class Vehicle {
      protected String brand;
      protected int year;
      protected double speed;
      
      public Vehicle(String brand, int year) {
          this.brand = brand;
          this.year = year;
          this.speed = 0;
      }
      
      public void start() {
          System.out.println(brand + " is starting...");
      }
      
      public void accelerate(double amount) {
          speed += amount;
          System.out.println("Speed increased to " + speed + " km/h");
      }
      
      public void displayInfo() {
          System.out.println(year + " " + brand + " (Speed: " + speed + " km/h)");
      }
  }
  
  // Derived class (subclass) - extends Vehicle
  class Car extends Vehicle {
      private int numberOfDoors;
      private boolean hasAirConditioning;
      
      public Car(String brand, int year, int numberOfDoors) {
          super(brand, year); // Call parent constructor
          this.numberOfDoors = numberOfDoors;
          this.hasAirConditioning = true;
      }
      
      // Override parent method
      @Override
      public void start() {
          System.out.println("Car engine started with key ignition");
          super.start(); // Call parent method
      }
      
      // Add new methods specific to Car
      public void honk() {
          System.out.println("Beep beep!");
      }
      
      public void toggleAC() {
          hasAirConditioning = !hasAirConditioning;
          System.out.println("Air conditioning " + (hasAirConditioning ? "ON" : "OFF"));
      }
      
      @Override
      public void displayInfo() {
          super.displayInfo();
          System.out.println("Doors: " + numberOfDoors + ", AC: " + hasAirConditioning);
      }
  }
  
  // Another derived class
  class Motorcycle extends Vehicle {
      private boolean hasSidecar;
      
      public Motorcycle(String brand, int year) {
          super(brand, year);
          this.hasSidecar = false;
      }
      
      @Override
      public void start() {
          System.out.println("Motorcycle started with kick start");
      }
      
      public void wheelie() {
          if (speed > 20) {
              System.out.println("Performing wheelie!");
          } else {
              System.out.println("Need more speed for wheelie");
          }
      }
  }
  
  public class ExtendsKeywordExample {
      public void demonstrateInheritance() {
          Car myCar = new Car("Toyota", 2022, 4);
          Motorcycle myBike = new Motorcycle("Honda", 2021);
          
          // Using inherited methods
          myCar.start();        // Uses overridden method
          myCar.accelerate(50); // Uses inherited method
          myCar.honk();         // Uses Car-specific method
          myCar.displayInfo();  // Uses overridden method
          
          System.out.println();
          
          myBike.start();        // Uses overridden method
          myBike.accelerate(30); // Uses inherited method
          myBike.wheelie();      // Uses Motorcycle-specific method
          myBike.displayInfo();  // Uses inherited method
      }
  }
  ```
  **Extraction**
  The process of retrieving specific information, such as dates, from a larger text.
  *Analogy: Like finding a needle in a haystack - you search through lots of text to pull out exactly what you need*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  import java.util.regex.Matcher;
  import java.util.regex.Pattern;
  import java.util.ArrayList;
  import java.util.List;
  
  public class ExtractionExample {
      public void demonstrateDataExtraction() {
          String text = "Our meeting is scheduled for 2024-03-15, and the deadline is 2024-03-22. " +
                       "Please confirm by 15/03/2024 or call us at +1-555-123-4567. " +
                       "Email: contact@company.com or visit www.company.com";
          
          System.out.println("Original text: " + text);
          System.out.println("\nExtracting information:");
          
          // Extract dates in ISO format (yyyy-MM-dd)
          extractISODates(text);
          
          // Extract dates in European format (dd/MM/yyyy)
          extractEuropeanDates(text);
          
          // Extract phone numbers
          extractPhoneNumbers(text);
          
          // Extract email addresses
          extractEmails(text);
          
          // Extract URLs
          extractURLs(text);
      }
      
      private void extractISODates(String text) {
          Pattern datePattern = Pattern.compile("\\d{4}-\\d{2}-\\d{2}");
          Matcher matcher = datePattern.matcher(text);
          
          System.out.println("\nISO Dates found:");
          while (matcher.find()) {
              String dateStr = matcher.group();
              try {
                  LocalDate date = LocalDate.parse(dateStr);
                  System.out.println("  " + dateStr + " -> " + date.format(DateTimeFormatter.ofPattern("MMMM dd, yyyy")));
              } catch (Exception e) {
                  System.out.println("  " + dateStr + " -> Invalid date");
              }
          }
      }
      
      private void extractEuropeanDates(String text) {
          Pattern datePattern = Pattern.compile("\\d{2}/\\d{2}/\\d{4}");
          Matcher matcher = datePattern.matcher(text);
          
          System.out.println("\nEuropean Dates found:");
          DateTimeFormatter europeanFormatter = DateTimeFormatter.ofPattern("dd/MM/yyyy");
          
          while (matcher.find()) {
              String dateStr = matcher.group();
              try {
                  LocalDate date = LocalDate.parse(dateStr, europeanFormatter);
                  System.out.println("  " + dateStr + " -> " + date.format(DateTimeFormatter.ofPattern("MMMM dd, yyyy")));
              } catch (Exception e) {
                  System.out.println("  " + dateStr + " -> Invalid date");
              }
          }
      }
      
      private void extractPhoneNumbers(String text) {
          Pattern phonePattern = Pattern.compile("\\+?1?-?\\(?\\d{3}\\)?-?\\d{3}-?\\d{4}");
          Matcher matcher = phonePattern.matcher(text);
          
          System.out.println("\nPhone numbers found:");
          while (matcher.find()) {
              System.out.println("  " + matcher.group());
          }
      }
      
      private void extractEmails(String text) {
          Pattern emailPattern = Pattern.compile("\\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Z|a-z]{2,}\\b");
          Matcher matcher = emailPattern.matcher(text);
          
          System.out.println("\nEmail addresses found:");
          while (matcher.find()) {
              System.out.println("  " + matcher.group());
          }
      }
      
      private void extractURLs(String text) {
          Pattern urlPattern = Pattern.compile("\\b(https?://|www\\.)\\S+");
          Matcher matcher = urlPattern.matcher(text);
          
          System.out.println("\nURLs found:");
          while (matcher.find()) {
              System.out.println("  " + matcher.group());
          }
      }
      
      // Extract specific information from structured text
      public void extractFromLog() {
          String logEntry = "[2024-03-15 14:30:22] ERROR: Failed to connect to database at 192.168.1.100:5432 - Connection timeout after 30 seconds";
          
          System.out.println("\nLog analysis:");
          System.out.println("Original log: " + logEntry);
          
          // Extract timestamp
          Pattern timestampPattern = Pattern.compile("\\[(.*?)\\]");
          Matcher timestampMatcher = timestampPattern.matcher(logEntry);
          if (timestampMatcher.find()) {
              System.out.println("Timestamp: " + timestampMatcher.group(1));
          }
          
          // Extract log level
          Pattern levelPattern = Pattern.compile("\\] (\\w+):");
          Matcher levelMatcher = levelPattern.matcher(logEntry);
          if (levelMatcher.find()) {
              System.out.println("Log Level: " + levelMatcher.group(1));
          }
          
          // Extract IP address
          Pattern ipPattern = Pattern.compile("(\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})");
          Matcher ipMatcher = ipPattern.matcher(logEntry);
          if (ipMatcher.find()) {
              System.out.println("IP Address: " + ipMatcher.group(1));
          }
          
          // Extract port number
          Pattern portPattern = Pattern.compile(":(\\d+)");
          Matcher portMatcher = portPattern.matcher(logEntry);
          if (portMatcher.find()) {
              System.out.println("Port: " + portMatcher.group(1));
          }
      }
  }
  ```

## F

  **File class**
  Represents a file or directory path, offering methods to create and delete files or directories and functionality to check their properties.
  *Analogy: Think of the File class like a map or address card. It doesn’t contain the house itself but tells you where the house (file/folder) is, and whether it exists.*
  ```Java
  import java.io.File;
  
  public class FileExample {
      public static void main(String[] args) {
          File myFile = new File("example.txt"); // Like having an address card
          
          if (myFile.exists()) {
              System.out.println("File exists at: " + myFile.getAbsolutePath());
          } else {
              System.out.println("File does not exist.");
          }
      }
  }
  
  ```
  **File handling**
  Essential part of programming that enables reading from and writing to files on a computer.
  *Analogy: File handling is like a mail system — reading letters from a mailbox (read) and putting letters inside (write).*
  ```java
  import java.io.*;
  
  public class FileHandlingExample {
      public static void main(String[] args) throws IOException {
          // Writing (sending a letter)
          FileWriter writer = new FileWriter("note.txt");
          writer.write("Hello, File Handling!");
          writer.close();
  
          // Reading (receiving a letter)
          BufferedReader reader = new BufferedReader(new FileReader("note.txt"));
          System.out.println("Message: " + reader.readLine());
          reader.close();
      }
  }
  
  ```
  **FileInputStream**
  Reads bytes from a file.
  *Analogy: Like using a straw to sip bytes of data (like juice) from a file.*
  ```java
  import java.io.FileInputStream;
  
  public class FileInputStreamExample {
      public static void main(String[] args) throws Exception {
          FileInputStream fis = new FileInputStream("input.txt");
          int data;
          while ((data = fis.read()) != -1) {
              System.out.print((char) data); // sip one byte at a time
          }
          fis.close();
      }
  }
  
  ```
  **FileOutputStream**
  Writes bytes to a file.
  *Analogy: Like pouring juice through a funnel into a bottle (file).*
  ```java
  import java.io.FileOutputStream;
  
  public class FileOutputStreamExample {
      public static void main(String[] args) throws Exception {
          FileOutputStream fos = new FileOutputStream("output.txt");
          String text = "Writing using FileOutputStream!";
          fos.write(text.getBytes()); // pour bytes into file
          fos.close();
      }
  }
  
  ```
  **FileReader**
  Class in Java that is used to read data from files in the form of characters.
  *Analogy: Like reading a book letter by letter instead of byte by byte.*
  ```java
  import java.io.FileReader;
  
  public class FileReaderExample {
      public static void main(String[] args) throws Exception {
          FileReader fr = new FileReader("characters.txt");
          int c;
          while ((c = fr.read()) != -1) {
              System.out.print((char) c); // read char by char
          }
          fr.close();
      }
  }
  
  ```
  **Files.createDirectories()**
  Method that creates the specified directory and any necessary parent directories if they don't already exist.
  *Analogy: Like asking a construction company to build a road with all missing stops along the way.*
  ```java
  import java.nio.file.*;
  
  public class CreateDirectoriesExample {
      public static void main(String[] args) throws Exception {
          Path path = Paths.get("parent/child/grandchild");
          Files.createDirectories(path); // build all missing folders
          System.out.println("Directories created: " + path.toAbsolutePath());
      }
  }
  
  ```
  **FileWriter**
  Class that handles file output, creates a file if it doesn’t exist, and overwrites existing files.
  *Analogy: Like writing a new page in a diary — it creates the diary if it doesn’t exist, and overwrites if it already does.*
  ```java
  import java.io.FileWriter;
  
  public class FileWriterExample {
      public static void main(String[] args) throws Exception {
          FileWriter fw = new FileWriter("diary.txt");
          fw.write("Dear Diary, today I learned FileWriter!");
          fw.close();
      }
  }
  
  ```
  **Final modifiers**
  Keyword in Java that makes a variable, method, or class immutable, so that its value or behavior cannot be modified once it has been initialized.
  *Analogy: Think of final like a permanent marker — once written, it cannot be changed.*
  ```java
  public class FinalExample {
      public static void main(String[] args) {
          final int MAX_USERS = 100; // written in permanent ink
          System.out.println("Max Users: " + MAX_USERS);
  
          // MAX_USERS = 200; // ❌ Error: cannot change final variable
      }
  }
  
  ```
  **First-In-First-Out (FIFO) principle**
  Method that means the first element added to the queue will be the first one removed.
  *Analogy: Like a line at a fast-food restaurant — the first person in line gets served first.*
  ```java
  import java.util.LinkedList;
  import java.util.Queue;
  
  public class FIFOExample {
      public static void main(String[] args) {
          Queue<String> queue = new LinkedList<>();
          queue.add("Alice");
          queue.add("Bob");
          queue.add("Carol");
  
          System.out.println(queue.poll()); // Alice (first in, first out)
          System.out.println(queue.poll()); // Bob
      }
  }
  
  ```
  **Format Style**
  The predefined styles in DateTimeFormatter for date and time formatting, such as SHORT, MEDIUM, LONG, and FULL.
  *Analogy: Like choosing a dress code (casual, business, formal) for how your date/time should look.*
  ```java
  import java.time.LocalDateTime;
  import java.time.format.DateTimeFormatter;
  import java.time.format.FormatStyle;
  
  public class FormatStyleExample {
      public static void main(String[] args) {
          LocalDateTime now = LocalDateTime.now();
  
          System.out.println(now.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT)));
          System.out.println(now.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM)));
          System.out.println(now.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG)));
          System.out.println(now.format(DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL)));
      }
  }
  
  ```
  **formatting**
  The act of converting a date or time object into a readable string format.
  *Analogy: Like putting a messy birthday cake into a nicely decorated box.*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  
  public class FormattingExample {
      public static void main(String[] args) {
          LocalDate today = LocalDate.now();
          String formattedDate = today.format(DateTimeFormatter.ofPattern("dd-MM-yyyy"));
          System.out.println("Formatted date: " + formattedDate);
      }
  }
  
  ```
  **full weekday name**
  A representation of the full name of the day, such as Monday, Tuesday, etc.
  *Analogy: Like saying “Monday” instead of just “Mon”.*
  ```java
  import java.time.LocalDate;
  import java.time.format.DateTimeFormatter;
  
  public class FullWeekdayExample {
      public static void main(String[] args) {
          LocalDate today = LocalDate.now();
          String dayName = today.format(DateTimeFormatter.ofPattern("EEEE"));
          System.out.println("Today is: " + dayName);
      }
  }
  
  ```
## G

  **get() method**
  Method used to retrieve values.
  *Analogy: Like grabbing a book from a specific shelf in a library.*
  ```java
  import java.util.ArrayList;
  
  public class GetMethodExample {
      public static void main(String[] args) {
          ArrayList<String> books = new ArrayList<>();
          books.add("Java Basics");
          books.add("OOP Concepts");
  
          System.out.println("First book: " + books.get(0)); // get() retrieves by index
      }
  }
  
  ```
  **Getter method**
  Enables other classes to read values.
  *Analogy: Like a receptionist who gives you information about an employee but doesn’t let you directly change it.*
  ```java
  class Person {
      private String name;
  
      public Person(String name) {
          this.name = name;
      }
  
      // Getter method
      public String getName() {
          return name;
      }
  }
  
  public class GetterExample {
      public static void main(String[] args) {
          Person p = new Person("Alice");
          System.out.println("Name: " + p.getName()); // use getter
      }
  }
  
  ```
  **getting current date**
  The use of LocalDate.now() to get the current date.
  *Analogy: Like checking the calendar on your phone for today’s date.*
  ```java
  import java.time.LocalDate;
  
  public class CurrentDateExample {
      public static void main(String[] args) {
          LocalDate today = LocalDate.now();
          System.out.println("Current date: " + today);
      }
  }
  
  ```
## H

  **HashMap**
  Allows null values and the data exists without a guaranteed order.
  *Analogy: Like a dictionary where each word (key) has a definition (value).*
  ```java
  import java.util.HashMap;
  
  public class HashMapExample {
      public static void main(String[] args) {
          HashMap<Integer, String> map = new HashMap<>();
          map.put(1, "Apple");
          map.put(2, "Banana");
          map.put(null, "Mango"); // null key allowed
  
          System.out.println(map.get(2)); // Banana
      }
  }
  
  ```
  **HashSet**
  Does not maintain any order and allows null values.
  *Analogy: Like a bag of unique marbles — no duplicates, no order.*
  ```java
  import java.util.HashSet;
  
  public class HashSetExample {
      public static void main(String[] args) {
          HashSet<String> set = new HashSet<>();
          set.add("Java");
          set.add("Python");
          set.add("Java"); // duplicate ignored
          set.add(null);
  
          System.out.println(set); // order not guaranteed
      }
  }
  ```
  **Hierarchical classification**
  Helps organize classes by having subclasses inherit from a common superclass, adding specific attributes.
  *Analogy: Like a family tree — one grandparent, many children with unique traits.*
  ```java
  class Animal {
      String type = "Animal";
  }
  
  class Dog extends Animal {
      String sound = "Bark";
  }
  
  class Cat extends Animal {
      String sound = "Meow";
  }
  
  public class HierarchicalClassification {
      public static void main(String[] args) {
          Dog d = new Dog();
          Cat c = new Cat();
          System.out.println("Dog is an " + d.type + " and says " + d.sound);
          System.out.println("Cat is an " + c.type + " and says " + c.sound);
      }
  }
  ```
  **Hierarchical inheritance**
  Enables multiple subclasses to inherit from a common superclass, making it efficient for shared functionality.
  *Analogy: Like a school principal (parent) with multiple teachers (children) inheriting school rules but adding their own subject knowledge.*
  ```java
  class Vehicle {
      void start() {
          System.out.println("Vehicle starting...");
      }
  }
  
  class Car extends Vehicle {
      void wheels() {
          System.out.println("Car has 4 wheels");
      }
  }
  
  class Bike extends Vehicle {
      void wheels() {
          System.out.println("Bike has 2 wheels");
      }
  }
  
  public class HierarchicalInheritance {
      public static void main(String[] args) {
          Car car = new Car();
          car.start();
          car.wheels();
  
          Bike bike = new Bike();
          bike.start();
          bike.wheels();
      }
  }
  ```
## I

  **import**
  A statement used in Java to include external classes or packages required for a program to run.
  *Analogy: Like asking the post office to bring you a package before you can use it.*
  ```java
  import java.util.ArrayList; // import statement
  
  public class ImportExample {
      public static void main(String[] args) {
          ArrayList<String> list = new ArrayList<>();
          list.add("Hello");
          System.out.println(list);
      }
  }
  ```
  **Import statement**
  Imports the File class from the java.io package, enabling file manipulation.
  *Analogy: Like unlocking the toolbox so you can use the hammer (File).*
  ```java
  import java.io.File; // import File class
  
  public class ImportStatementExample {
      public static void main(String[] args) {
          File myFile = new File("test.txt");
          System.out.println("Path: " + myFile.getAbsolutePath());
      }
  }
  ```
  **Inheritance**
  A way to create new classes based on existing classes. The concept in which one class, known as the child, derives properties and methods from another class, the parent.
  *Analogy: Like a child inheriting eye color from a parent.*
  ```java
  class Parent {
      String familyName = "Smith";
  }
  
  class Child extends Parent {
      String firstName = "Alice";
  }
  
  public class InheritanceExample {
      public static void main(String[] args) {
          Child c = new Child();
          System.out.println(c.firstName + " " + c.familyName);
      }
  }
  ```
  **Initialization**
  Assigns values to the object’s attributes.
  *Analogy: Like filling in the blanks when creating a new ID card.*
  ```java
  class Student {
      String name;
      int age;
  
      Student(String n, int a) { // initialization via constructor
          name = n;
          age = a;
      }
  }
  
  public class InitializationExample {
      public static void main(String[] args) {
          Student s = new Student("Bob", 20); // initialized
          System.out.println(s.name + " is " + s.age + " years old");
      }
  }
  ```
  **Inner class**
  Defined within another class in Java. Helps organize code better.
  *Analogy: Like a department inside a company — it’s inside but part of the bigger system.*
  ```java
  class Outer {
      class Inner {
          void show() {
              System.out.println("This is an inner class");
          }
      }
  }
  
  public class InnerClassExample {
      public static void main(String[] args) {
          Outer.Inner obj = new Outer().new Inner();
          obj.show();
      }
  }
  ```
  **InputStream**
  Abstract class representing an input stream of bytes. It serves as the superclass for all classes that read byte data.
  *Analogy: Like a pipeline bringing bytes into your program.*
  ```java
  import java.io.FileInputStream;
  import java.io.InputStream;
  
  public class InputStreamExample {
      public static void main(String[] args) throws Exception {
          InputStream is = new FileInputStream("input.txt");
          int data;
          while ((data = is.read()) != -1) {
              System.out.print((char) data);
          }
          is.close();
      }
  }
  ```
  **Instant**
  A class representing a moment on the timeline in UTC.
  *Analogy: Like taking an exact timestamped photo of a moment in UTC.*
  ```java
  import java.time.Instant;
  
  public class InstantExample {
      public static void main(String[] args) {
          Instant now = Instant.now();
          System.out.println("Current Instant: " + now);
      }
  }
  ```
  **Instantiation**
  Creates a new object.
  *Analogy: Like actually building a car from its blueprint (class).*
  ```java
  class Car {
      String brand = "Toyota";
  }
  
  public class InstantiationExample {
      public static void main(String[] args) {
          Car myCar = new Car(); // instantiation
          System.out.println("Car brand: " + myCar.brand);
      }
  }
  ```
  **Interface**
  Reference type that defines a contract for classes to follow. It can contain constants, method signatures, default methods, static methods, and nested types but cannot have instance fields or constructors.
  *Analogy: Like a job contract — it tells what work must be done, but not how.*
  ```java
  interface Animal {
      void sound();
  }
  
  class Dog implements Animal {
      public void sound() {
          System.out.println("Bark");
      }
  }
  
  public class InterfaceExample {
      public static void main(String[] args) {
          Animal d = new Dog();
          d.sound(); // Bark
      }
  }
  
  ```
  **ISO-8601**
  A standard for representing date and time in an international format.
  *Analogy: Like a universal date format everyone agrees on (YYYY-MM-DD).*
  ```java
  import java.time.LocalDateTime;

  public class ISO8601Example {
      public static void main(String[] args) {
          LocalDateTime now = LocalDateTime.now();
          System.out.println("ISO-8601 Format: " + now); // default ISO-8601
      }
  }

  ```
## J

  **Java 8 Time API**
  A set of classes introduced in Java 8 under the java.time package for date and time manipulation.
  *Analogy: Like upgrading from an old wall clock to a modern smartwatch — more precise and feature-rich.*
  ```java
  import java.time.*;
  
  public class Java8TimeAPI {
      public static void main(String[] args) {
          LocalDate today = LocalDate.now();
          LocalTime time = LocalTime.now();
          LocalDateTime dateTime = LocalDateTime.now();
  
          System.out.println("Date: " + today);
          System.out.println("Time: " + time);
          System.out.println("DateTime: " + dateTime);
      }
  }
  ```
  **Java Collections Framework (JCF)**
  Powerful Java tool that allows developers to work with groups of objects. The JCF provides a set of classes and interfaces to handle data collections based on developers' needs.
  *Analogy: Like a toolbox with different containers (List, Set, Map) for organizing items.*
  ```java
  import java.util.*;
  
  public class JCFExample {
      public static void main(String[] args) {
          List<String> list = new ArrayList<>();
          list.add("Java");
          list.add("Python");
  
          Set<String> set = new HashSet<>(list);
          Map<Integer, String> map = new HashMap<>();
          map.put(1, "Java");
          map.put(2, "C++");
  
          System.out.println("List: " + list);
          System.out.println("Set: " + set);
          System.out.println("Map: " + map);
      }
  }
  ```
  **Java new input or output (NIO)**
  Offers an improved and more efficient way to handle file and directory operations compared to the traditional Java IO.
  *Analogy: Like switching from a paper letter system (old I/O) to instant messaging (NIO) for faster operations.*
  ```java
  import java.nio.file.*;
  
  public class NIOExample {
      public static void main(String[] args) throws Exception {
          Path path = Paths.get("example.txt");
          Files.write(path, "Hello NIO!".getBytes()); // write
          String content = new String(Files.readAllBytes(path)); // read
          System.out.println(content);
      }
  }
  ```
## K

  **keySet()**
  Used to iterate keys.
  *Analogy: Like looking at just the labels on drawers without opening them.*
  ```java
  import java.util.HashMap;
  
  public class KeySetExample {
      public static void main(String[] args) {
          HashMap<Integer, String> map = new HashMap<>();
          map.put(1, "Red");
          map.put(2, "Blue");
  
          for (Integer key : map.keySet()) {
              System.out.println("Key: " + key);
          }
      }
  }
  ```
## L

  **LinkedList**
  Uses a doubly linked list to store elements. This array type efficiently adds and removes elements but operates slowly when accessing elements by index.
  *Analogy: Like a train of carriages — easy to add/remove but harder to jump directly to the 5th carriage.*
  ```java
  import java.util.LinkedList;
  
  public class LinkedListExample {
      public static void main(String[] args) {
          LinkedList<String> list = new LinkedList<>();
          list.add("A");
          list.add("B");
          list.addFirst("Start");
          list.addLast("End");
          System.out.println(list);
      }
  }
  ```
  **List**
  Ordered collection that allows duplicate elements and maintains order.
  *Analogy: Like a grocery list — ordered and can have duplicates.*
  ```java
  import java.util.List;
  import java.util.ArrayList;
  
  public class ListExample {
      public static void main(String[] args) {
          List<String> fruits = new ArrayList<>();
          fruits.add("Apple");
          fruits.add("Apple"); // duplicates allowed
          fruits.add("Banana");
          System.out.println(fruits);
      }
  }
  ```
  **list directory**
  Operation that enables users to view the contents of a specified directory. It retrieves the names of all the files and subdirectories within that directory, helping users quickly check what is stored inside and navigate their files more efficiently.
  *Analogy: Like opening a folder on your computer to see what’s inside.*
  ```java
  import java.io.File;
  
  public class ListDirectoryExample {
      public static void main(String[] args) {
          File dir = new File(".");
          String[] files = dir.list();
          for (String f : files) {
              System.out.println(f);
          }
      }
  }
  ```
  **list()**
  Method that can retrieve an array of strings representing the names of files and directories.
  *Analogy: Like asking for a name-only summary of what’s inside a folder.*
  ```java
  import java.io.File;
  
  public class ListMethodExample {
      public static void main(String[] args) {
          File dir = new File(".");
          String[] names = dir.list();
          for (String name : names) {
              System.out.println("File: " + name);
          }
      }
  }
  ```
  **LocalDate**
  A Java class that represents a date without time-zone information.
  *Analogy: Like a wall calendar date without time or zone.*
  ```java
  import java.time.LocalDate;
  
  public class LocalDateExample {
      public static void main(String[] args) {
          LocalDate date = LocalDate.of(2025, 8, 17);
          System.out.println(date);
      }
  }
  ```
  **LocalDate.now()**
  A method that retrieves the current date from the system clock.
  *Analogy: Like checking today’s date on your phone.*
  ```java
  import java.time.LocalDate;
  
  public class LocalDateNow {
      public static void main(String[] args) {
          System.out.println(LocalDate.now());
      }
  }
  
  ```
  **LocalDate.of()**
  A method used to create a LocalDate object with a specific year, month, and day.
  *Analogy: Like manually writing a date on a form.*
  ```java
  import java.time.LocalDate;
  
  public class LocalDateOf {
      public static void main(String[] args) {
          LocalDate date = LocalDate.of(2024, 12, 25);
          System.out.println("Christmas: " + date);
      }
  }
  ```
  **LocalDate.parse()**
  A method in Java used to convert a string into a LocalDate object using a specific pattern.
  *Analogy: Like converting a text date string into an actual calendar date.*
  ```java
  import java.time.LocalDate;
  
  public class LocalDateParse {
      public static void main(String[] args) {
          LocalDate date = LocalDate.parse("2025-08-17");
          System.out.println(date);
      }
  }
  ```
  **LocalDateTime**
  A Java class representing both date and time without time-zone information.
  *Analogy: Like writing down date + time but ignoring the time zone.*
  ```java
  import java.time.LocalDateTime;
  
  public class LocalDateTimeExample {
      public static void main(String[] args) {
          LocalDateTime now = LocalDateTime.now();
          System.out.println(now);
      }
  }
  ```
  **LocalTime**
  A class representing a time that includes an hour, minute, and second without a date.
  *Analogy: Like looking at a digital clock without date.*
  ```java
  import java.time.LocalTime;
  
  public class LocalTimeExample {
      public static void main(String[] args) {
          LocalTime now = LocalTime.now();
          System.out.println(now);
      }
  }
  ```
  **Loop**
  Function that reads each line until there are no more lines, printing each line to the console.
  *Analogy: Like flipping through a stack of papers one by one until none left.*
  ```java
  import java.io.*;
  
  public class LoopExample {
      public static void main(String[] args) throws Exception {
          BufferedReader br = new BufferedReader(new FileReader("data.txt"));
          String line;
          while ((line = br.readLine()) != null) {
              System.out.println(line); // print each line
          }
          br.close();
      }
  }
  ```
## M

  **Main class**
  Exhibits implementation of the abstract method and inherits concrete behavior from an abstract class.
  *Analogy: Think of an architect’s blueprint (abstract class). The builder (main class) must implement the missing details to complete the house.*
  ```java
  abstract class Shape {
      abstract void draw(); // Abstract method
      void info() {
          System.out.println("This is a shape");
      }
  }
  
  public class Main extends Shape {
      @Override
      void draw() {
          System.out.println("Drawing a Circle");
      }
  
      public static void main(String[] args) {
          Main obj = new Main();
          obj.info();
          obj.draw();
      }
  }
  ```
  **main() method**
  Method that indicates the start of program execution.
  *Analogy: Like pressing the power button on a machine—it’s where everything begins.*
  ```java
  public class Main {
      public static void main(String[] args) {
          System.out.println("Program started!");
      }
  }
  
  ```
  **Map**
  Unordered collection of key-value pairs where each key is unique.
  *Analogy:
  Like a dictionary—each word (key) maps to its meaning (value).*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Map<Integer, String> students = new HashMap<>();
          students.put(1, "Alice");
          students.put(2, "Bob");
          students.put(3, "Charlie");
  
          System.out.println(students.get(2)); // Bob
      }
  }
  ```
  **Method call**
  Process of invoking a method on an object or class to execute its defined behavior.
  *Analogy: Calling a friend on the phone—you "invoke" them, and they "respond."*
  ```java
  class Calculator {
      int add(int a, int b) {
          return a + b;
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Calculator calc = new Calculator();
          int result = calc.add(5, 3); // Method call
          System.out.println("Sum = " + result);
      }
  }
  ```
  **Method overriding**
  Allows the child to perform specific tasks in its own way, making its behavior more flexible and dynamic.
  *Analogy: A parent says, "Go to school by bus." The child says, "No, I’ll ride my bike instead." Same rule, different execution.*
  ```java
  class Animal {
      void sound() {
          System.out.println("Animal makes a sound");
      }
  }
  
  class Dog extends Animal {
      @Override
      void sound() {
          System.out.println("Dog barks");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Animal myDog = new Dog();
          myDog.sound(); // Dog barks
      }
  }
  ```
  **Method signature**
  Allows only method declarations without bodies and provides a blueprint for implementing classes.
  *Analogy: Like a job title "Doctor (MBBS, MD)"—it describes qualifications but doesn’t explain what they actually do.*
  ```java
  interface Vehicle {
      void start(); // Method signature (no body)
  }
  ```
  **Method-local inner class**
  Defined within a method of the outer class. Method-only local classes can only be accessed within that method.
  *Analogy: Like a temporary assistant hired only for one project.*
  ```java
  public class Main {
      void outerMethod() {
          class Inner {
              void show() {
                  System.out.println("Inside method-local inner class");
              }
          }
          Inner obj = new Inner();
          obj.show();
      }
  
      public static void main(String[] args) {
          Main main = new Main();
          main.outerMethod();
      }
  }
  ```
  **Methods**
  Actions performed by an object.
  *Analogy: A remote control (object) can "turn on TV" or "increase volume" → those actions are methods.*
  ```java
  class Car {
      void drive() {
          System.out.println("The car is driving");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Car myCar = new Car();
          myCar.drive(); // Method call
      }
  }
  ```
  **mkdirs()**
  Method that creates the directory along with any missing parent directories. The program checks if the directory already exists before attempting to create it.
  *Analogy: If you want a folder inside another folder, mkdirs() creates all necessary parent folders if they don’t exist.*
  ```java
  import java.io.File;
  
  public class Main {
      public static void main(String[] args) {
          File dir = new File("C:/example/newfolder/subfolder");
          if (dir.mkdirs()) {
              System.out.println("Directories created successfully!");
          } else {
              System.out.println("Failed to create directories or already exist.");
          }
      }
  }
  ```
  **MonthDay**
  A class that represents a combination of a month and day without a year.
  *Analogy: Like a birthday—always the same month and day, regardless of the year.*
  ```java
  import java.time.MonthDay;
  
  public class Main {
      public static void main(String[] args) {
          MonthDay birthday = MonthDay.of(12, 25); // Dec 25
          System.out.println("Birthday: " + birthday);
      }
  }
  ```
  **Multilevel inheritance**
  Allows inheritance from other subclasses.
  *Analogy: Grandparent → Parent → Child (traits pass down generations).*
  ```java
  class Grandparent {
      void greet() {
          System.out.println("Hello from Grandparent");
      }
  }
  
  class Parent extends Grandparent {
      void guide() {
          System.out.println("Guidance from Parent");
      }
  }
  
  class Child extends Parent {
      void play() {
          System.out.println("Child is playing");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Child c = new Child();
          c.greet();
          c.guide();
          c.play();
      }
  }
  ```
  **Multiple inheritance**
  Indicates the process of one subclass inheriting properties from multiple superclasses.
  *Analogy: A student can be both a musician and an athlete (interfaces).*
  ```java
  interface Musician {
      void playMusic();
  }
  
  interface Athlete {
      void playSport();
  }
  
  class Student implements Musician, Athlete {
      public void playMusic() {
          System.out.println("Student plays guitar");
      }
      public void playSport() {
          System.out.println("Student plays football");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Student s = new Student();
          s.playMusic();
          s.playSport();
      }
  }
  ```
## N

  **newline()**
  Method that adds a new line.
  *Analogy:
  Like pressing the Enter key when typing.*
  ```java
  public class Main {
      public static void main(String[] args) {
          System.out.print("Hello");
          System.out.println(); // newline
          System.out.print("World");
      }
  }
  ```
  **No-arg constructor**
  Type of default constructor that does not accept any arguments or parameters. It allows the creation of an object without requiring any initial values to be provided by the user.
  *Analogy: When you buy a new phone, it comes with default settings if you don’t customize it.*
  ```java
  class Car {
      Car() { // No-arg constructor
          System.out.println("Car created with default settings");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Car myCar = new Car();
      }
  }
  ```
  **Non-access modifiers**
  Keywords that add extra details about elements, influencing their behavior while ensuring access levels remain unaffected.
  *Analogy: Like adding “electric” or “manual” to a car—it changes how it works but not who can use it.*
  ```java
  class Main {
      final int MAX = 100; // final modifier
      static void greet() { // static modifier
          System.out.println("Hello!");
      }
  
      public static void main(String[] args) {
          greet();
      }
  }
  ```
  **Non-static inner class**
  Can access both static and non-static members of the outer class.
  *Analogy: Like a child who has access to both the family house (static) and personal belongings (non-static).*
  ```java
  class Outer {
      private String msg = "Hello from Outer";
  
      class Inner {
          void display() {
              System.out.println(msg); // Can access outer members
          }
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Outer outer = new Outer();
          Outer.Inner inner = outer.new Inner();
          inner.display();
      }
  }
  ```
## O

  **Object**
  Instance of a class that represents a real-world entity or concept.
  *Analogy: The class is a mold, the object is the actual product created from it.*
  ```java
  class Dog {
      String name;
      Dog(String name) {
          this.name = name;
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Dog myDog = new Dog("Buddy"); // Object
          System.out.println("Dog's name: " + myDog.name);
      }
  }
  ```
  **OffsetDateTime**
  A date-time with an offset from UTC/Greenwich.
  *Analogy: Like saying "8:00 AM UTC+5:30" instead of just "8:00 AM".*
  ```java
  import java.time.OffsetDateTime;
  import java.time.ZoneOffset;
  
  public class Main {
      public static void main(String[] args) {
          OffsetDateTime dateTime = OffsetDateTime.now(ZoneOffset.ofHours(5));
          System.out.println(dateTime);
      }
  }
  ```
  **Organized code**
  Structures classes into hierarchies, making it easier to understand and extend functionality.
  *Analogy: Like arranging a library into sections: fiction, science, history → makes it easier to find books.*
  ```java
  // Package structure for organized code
  package vehicles;
  
  class Car {
      void drive() {
          System.out.println("Car is driving");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Car c = new Car();
          c.drive();
      }
  }
  ```
  **OutputStream**
  Abstract class representing an output stream of bytes. It serves as the superclass for all classes that write byte data.
  *Analogy: Like a pipeline that carries water (bytes) to different destinations.*
  ```java
  import java.io.FileOutputStream;
  import java.io.OutputStream;
  
  public class Main {
      public static void main(String[] args) {
          try {
              OutputStream os = new FileOutputStream("output.txt");
              os.write("Hello World".getBytes());
              os.close();
              System.out.println("Data written to file.");
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```
## P

  **Parameterized constructor**
  Allows values to be passed to an object at the time of creation, enabling specific initialization of the object’s attributes.
  *Analogy: Like ordering a pizza with specific toppings instead of default/plain.*
  ```java
  class Car {
      String model;
      int year;
  
      Car(String model, int year) { // Parameterized constructor
          this.model = model;
          this.year = year;
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Car car = new Car("Tesla", 2025);
          System.out.println(car.model + " - " + car.year);
      }
  }
  ```
  **Paths.get()**
  Creates a Path object representing the directory path. This Path object can then be used in various operations like reading, writing, or creating directories.
  *Analogy: Like typing an address into Google Maps before navigating.*
  ```java
  import java.nio.file.*;
  
  public class Main {
      public static void main(String[] args) {
          Path path = Paths.get("C:/Users/Example/Documents/file.txt");
          System.out.println("Path: " + path);
      }
  }
  ```
  **Polymorphism**
  Process that allows objects to share the same interface while exhibiting different behaviors based on their specific implementations. This process allows objects to be treated as instances of their parent class, meaning different objects can be accessed through the same interface.
  *Analogy: The word "drive" means different things: Car drives, Bike drives, Airplane drives (takeoff).*
  ```java
  class Vehicle {
      void move() { System.out.println("Vehicle moves"); }
  }
  
  class Car extends Vehicle {
      @Override
      void move() { System.out.println("Car drives on road"); }
  }
  
  class Plane extends Vehicle {
      @Override
      void move() { System.out.println("Plane flies in sky"); }
  }
  
  public class Main {
      public static void main(String[] args) {
          Vehicle v1 = new Car();
          Vehicle v2 = new Plane();
          v1.move();
          v2.move();
      }
  }
  ```
  **Print spooling**
  Process wherein print jobs are placed in a queue and processed in order.
  *Analogy: Like people waiting in line to use a printer.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Queue<String> printQueue = new LinkedList<>();
          printQueue.add("Job1");
          printQueue.add("Job2");
          printQueue.add("Job3");
  
          while (!printQueue.isEmpty()) {
              System.out.println("Printing: " + printQueue.poll());
          }
      }
  }
  ```
  **Private modifier**
  Restricts access to an element, allowing it to be accessed only within the same class where it is declared.
  *Analogy: Like your personal diary—only you can read it.*
  ```java
  class Person {
      private String name = "Secret";
  
      private void showName() {
          System.out.println(name);
      }
  }
  ```
  **Properties**
  Characteristics or attributes of an object.
  *Analogy: A phone has properties like brand, color, battery life.*
  ```java
  class Book {
      String title;
      String author;
  }
  
  public class Main {
      public static void main(String[] args) {
          Book b = new Book();
          b.title = "1984";
          b.author = "George Orwell";
          System.out.println(b.title + " by " + b.author);
      }
  }
  ```
  **Protected modifier**
  Allows access to a class member within the same package and by subclasses, even in different packages.
  *Analogy: Like a family recipe—accessible to family members (subclasses), but not outsiders.*
  ```java
  class Parent {
      protected void show() {
          System.out.println("Protected method");
      }
  }
  
  class Child extends Parent {
      void display() {
          show(); // Accessible
      }
  }
  ```
  **Public modifier**
  Allows elements such as classes, methods, or variables to be accessible from any other class in any package, making the element globally accessible throughout the program.
  *Analogy: Like a public park—anyone can enter.*
  ```java
  public class Car {
      public void drive() {
          System.out.println("Driving...");
      }
  }
  ```
  **put() method**
  Used to add entries.
  *Analogy: Like putting items into labeled boxes.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Map<Integer, String> map = new HashMap<>();
          map.put(1, "Apple");
          map.put(2, "Banana");
          System.out.println(map);
      }
  }
  ```
## Q

  **Queue**
  Collection of elements that offers two main operations, enqueue and dequeue.
  *Analogy: Like a line at a ticket counter.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Queue<String> q = new LinkedList<>();
          q.add("Alice");
          q.add("Bob");
          q.add("Charlie");
  
          System.out.println(q.poll()); // Alice
      }
  }
  ```
## R

  **remove() method**
  Used to remove entries.
  *Analogy: Like erasing an item from a to-do list.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Map<Integer, String> map = new HashMap<>();
          map.put(1, "Apple");
          map.put(2, "Banana");
          map.remove(1); // Removes Apple
          System.out.println(map);
      }
  }
  ```
  **Runtime polymorphism**
  Occurs when a subclass provides its specific implementation of a method defined in its superclass. It is also known as method overriding.
  *Analogy: Saying "play" — a child plays with toys, a musician plays music. Same word, different meaning.*
  ```java
  class Animal {
      void sound() { System.out.println("Animal makes sound"); }
  }
  
  class Dog extends Animal {
      @Override
      void sound() { System.out.println("Dog barks"); }
  }
  
  public class Main {
      public static void main(String[] args) {
          Animal a = new Dog(); // Runtime decision
          a.sound();
      }
  }
  ```
## S

  **Set**
  Unordered collection that does not allow duplicate elements.
  *Analogy: Like a basket of unique fruits—you can’t put the same fruit twice.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          Set<String> set = new HashSet<>();
          set.add("Apple");
          set.add("Banana");
          set.add("Apple"); // ignored
          System.out.println(set);
      }
  }
  ```
  **Setter method**
  Allows modification with validation.
  *Analogy: Like a gatekeeper who lets only valid people enter.*
  ```java
  class Person {
      private int age;
  
      public void setAge(int age) {
          if (age > 0) this.age = age; // validation
      }
  
      public int getAge() { return age; }
  }
  ```
  **Single inheritance**
  Allows a subclass to inherit from just one superclass, simplifying the structure.
  *Analogy: Like a child having one biological parent in terms of traits inheritance.*
  ```java
  class Parent {
      void greet() { System.out.println("Hello from Parent"); }
  }
  
  class Child extends Parent {
      void speak() { System.out.println("Hi, I’m Child"); }
  }
  ```
  **Static modifier**
  Implies that a member belongs to the class, not to instances, allowing access without creating an object.
  *Analogy: Like a country’s national anthem—it belongs to everyone, not one person.*
  ```java
  class MathUtil {
      static int square(int n) { return n * n; }
  }
  
  public class Main {
      public static void main(String[] args) {
          System.out.println(MathUtil.square(5));
      }
  }
  ```
  **Static nested class**
  Cannot access non-static members of the outer class directly. Static nested classes are associated with the outer class but do not require an instance of the outer class for instantiation.
  *Analogy: Like a static department inside a company—exists independently of specific employees.*
  ```java
  class Outer {
      static class Inner {
          void show() { System.out.println("Static Nested Class"); }
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Outer.Inner obj = new Outer.Inner();
          obj.show();
      }
  }
  ```
  **Subclass**
  Inherits properties and methods from a larger class known as a superclass, allowing the creation of a new class with added or modified features.
  *Analogy: A smartphone is a subclass of "Phone" with added features like apps.*
  ```java
  class Vehicle {
      void move() { System.out.println("Moving..."); }
  }
  
  class Car extends Vehicle {
      void honk() { System.out.println("Beep Beep!"); }
  }
  ```
  **Superclass**
  Class whose properties and methods are inherited by another class.
  *Analogy: Parent in a family tree.*
  ```java
  class Animal { // Superclass
      void eat() { System.out.println("Eating..."); }
  }
  class Dog extends Animal { } // Subclass
  ```
## T

  **TreeMap**
  Allows null values and maintains keys in a sorted order.
  *Analogy: Like an alphabetically arranged dictionary.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          TreeMap<Integer, String> map = new TreeMap<>();
          map.put(3, "C");
          map.put(1, "A");
          map.put(2, "B");
          System.out.println(map); // Sorted by keys
      }
  }
  ```
  **TreeSet**
  Maintains a sorted order of elements, and does not allow null values.
  *Analogy: Like arranging books on a shelf by alphabetical order, no duplicates.*
  ```java
  import java.util.*;
  
  public class Main {
      public static void main(String[] args) {
          TreeSet<String> set = new TreeSet<>();
          set.add("Banana");
          set.add("Apple");
          set.add("Apple"); // ignored
          System.out.println(set);
      }
  }
  ```
  **try block**
  Block of code that is used for exception handling, ensuring that errors during reading are caught.
  *Analogy: Like airbags in a car—they don’t stop the accident, but protect you.*
  ```java
  public class Main {
      public static void main(String[] args) {
          try {
              int x = 5 / 0;
          } catch (Exception e) {
              System.out.println("Error: " + e.getMessage());
          }
      }
  }
  ```
## U

  **UTC**
  A coordinated universal time standard used globally.
  *Analogy: Like Greenwich as the zero milestone for world time.*
  ```java
  import java.time.*;
  
  public class Main {
      public static void main(String[] args) {
          Instant now = Instant.now();
          System.out.println("UTC Time: " + now);
      }
  }
  
  ```
## V

  **Virtual method**
  Any method that can be overridden in a subclass, enabling dynamic method resolution.
  *Analogy: Like a template recipe—children can modify ingredients.*
  ```java
  class Animal {
      void speak() { System.out.println("Animal sound"); }
  }
  
  class Cat extends Animal {
      @Override
      void speak() { System.out.println("Meow"); }
  }
  
  ```
## W

  **write() method**
  Method that writes data to a file.
  *Analogy: Like writing notes into a notebook.*
  ```java
  import java.io.*;
  
  public class Main {
      public static void main(String[] args) {
          try {
              FileWriter fw = new FileWriter("data.txt");
              fw.write("Hello File!");
              fw.close();
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  ```
## Y

  **Year**
  A class that represents a year, such as 2025.
  *Analogy: Like seeing "2025" printed on a calendar header.*
  ```java
  import java.time.Year;
  
  public class Main {
      public static void main(String[] args) {
          Year y = Year.of(2025);
          System.out.println(y);
      }
  }
  
  ```
  **YearMonth**
  A class that represents a combination of a year and month without a day.
  *Analogy: Like "August 2025" on a credit card expiry date.*
  ```java
  import java.time.YearMonth;
  
  public class Main {
      public static void main(String[] args) {
          YearMonth ym = YearMonth.of(2025, 8);
          System.out.println(ym);
      }
  }
  ```
## Z

  **ZonedDateTime**
  A class representing a date and time with a time zone.
  *Analogy: Like writing "10 AM, New York Time" in a schedule.*
  ```java
  import java.time.*;
  
  public class Main {
      public static void main(String[] args) {
          ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
          System.out.println(zdt);
      }
  }
  ```
  **ZoneId**
  An identifier for a time zone, such as "Asia/Kolkata."
  *Analogy: Like a label "Asia/Kolkata" or "America/New_York" on world clocks.*
  ```java
  import java.time.*;
  
  public class Main {
      public static void main(String[] args) {
          ZoneId zone = ZoneId.of("Asia/Kolkata");
          System.out.println(zone);
      }
  }
  ```
  **ZoneOffset**
  The amount of time added or subtracted from UTC to get local time.
  *Analogy: Like saying "IST = UTC+5:30".*
  ```java
  import java.time.*;
  
  public class Main {
      public static void main(String[] args) {
          ZoneOffset offset = ZoneOffset.ofHoursMinutes(5, 30);
          System.out.println("Offset: " + offset);
      }
  }
  
  ```

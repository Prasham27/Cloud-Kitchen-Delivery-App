
# Cloud Kitchen Food Delivery App Database

This project provides a comprehensive database schema for a **Cloud Kitchen Food Delivery System**. The system is designed to facilitate seamless interaction between customers, restaurant owners, delivery agents, and the platform admin, all without the need for a physical dine-in space. It enables end-to-end management of food orders, reward programs, restaurant menus, delivery logistics, and financial operations.

---

## Key Features

This database is engineered to support a wide range of features tailored for a modern food delivery platform:

- **Customer & Order Management**: Handles user registration, food ordering, order tracking, and reward coin systems. Maintains customer profiles, order history, and ratings.

- **Restaurant & Menu Management**: Allows cloud kitchen owners to register their restaurants, manage menus, pricing, ingredients, and monitor reviews.

- **Delivery Agent Management**: Supports agent onboarding, order delivery tracking, earnings management, and payout requests.

- **Admin & Platform Oversight**: Empowers the platform owner to approve restaurants, manage disputes, control commissions, and monitor all system activities.

- **Reward Coin System**: Encourages customer loyalty by allowing coin accumulation on orders and redemption under set rules.

- **Financial Reporting**: Tracks all transactions including platform commissions, restaurant revenue, and delivery agent payouts.

- **Search & Discovery**: Enables customers and visitors to explore restaurants and dishes using filters like cuisine, ingredients, and ratings.

- **Secure, Role-Based Access**: Provides different dashboards and privileges to various user roles ensuring data privacy and control.

---

## Application Users

The system supports a wide range of users across the food delivery ecosystem:

- **Visitors**: Can browse restaurants and menus without registering.
- **Customers**: Registered users who place orders and earn/redeem reward coins.
- **Restaurant Owners**: Manage their cloud kitchens, menus, and orders.
- **Delivery Agents**: Handle assigned deliveries and track earnings.
- **Admin (Platform Owner)**: Oversees registrations, payments, compliance, and overall platform analytics.

---

## Core Use Cases

### Visitors
- Browse menus, restaurants, and reviews.
- Search by cuisine, dish name, or ingredients.

### Customers
- Register/login, manage profile.
- Browse/search menu items.
- Add to cart, place orders, and track them.
- Earn/redeem reward coins.
- Rate restaurants and agents.
- View past orders and re-order.

### Restaurant Owners
- Register/login their restaurant.
- Manage restaurant and menu details.
- Accept and prepare orders.
- View earnings, commissions, and customer feedback.

### Delivery Agents
- Register/login to the system.
- Manage assigned deliveries and update status.
- Track earnings and request payouts.

### Admin (Platform Owner)
- Approve/reject restaurants, verify licenses.
- Manage all users and accounts.
- Monitor orders, disputes, and platform revenue.
- Handle reward coins and system-wide reports.

---

## Database Schema

The database is composed of several interconnected tables that model the various entities and relationships within the cloud kitchen food delivery ecosystem.

```sql
-- Create Users table 
CREATE TABLE Users ( 
    UserID INT PRIMARY KEY, 
    Name VARCHAR(100) NOT NULL, 
    Email VARCHAR(100) UNIQUE NOT NULL, 
    Password VARCHAR(255) NOT NULL, 
    PhoneNumber VARCHAR(20), 
    Address TEXT, 
    UserType VARCHAR(20) NOT NULL CHECK (UserType IN ('Customer', 'RestaurantOwner', 'DeliveryAgent')) 
); 

-- Create Customer table 
CREATE TABLE Customer ( 
    CustomerID INT PRIMARY KEY, 
    UserID INT UNIQUE NOT NULL, 
    RewardCoins INT DEFAULT 0, 
    PaymentPreferences VARCHAR(100), 
    FOREIGN KEY (UserID) REFERENCES Users(UserID) ON DELETE CASCADE 
); 

-- Create RestaurantOwner table 
CREATE TABLE RestaurantOwner ( 
    RestaurantOwnerID INT PRIMARY KEY, 
    UserID INT UNIQUE NOT NULL, 
    LicenseNumber VARCHAR(50) NOT NULL, 
    FOREIGN KEY (UserID) REFERENCES Users(UserID) ON DELETE CASCADE 
); 

-- Create DeliveryAgent table 
CREATE TABLE DeliveryAgent ( 
    DeliveryAgentID INT PRIMARY KEY, 
    UserID INT UNIQUE NOT NULL, 
    Earnings DECIMAL(10, 2) DEFAULT 0.00, 
    FOREIGN KEY (UserID) REFERENCES Users(UserID) ON DELETE CASCADE 
); 

-- Create Restaurant table 
CREATE TABLE Restaurant ( 
    RestaurantID INT PRIMARY KEY, 
    Name VARCHAR(100) NOT NULL, 
    Address TEXT NOT NULL, 
    ContactInfo VARCHAR(100), 
    CuisineType VARCHAR(50), 
    AverageRating DECIMAL(3, 2) DEFAULT 0.00, 
    LicenseNumber VARCHAR(50) NOT NULL, 
    RestaurantOwnerID INT NOT NULL, 
    FOREIGN KEY (RestaurantOwnerID) REFERENCES RestaurantOwner(RestaurantOwnerID) ON DELETE CASCADE 
); 

-- Create Menu table 
CREATE TABLE Menu ( 
    MenuID INT PRIMARY KEY, 
    RestaurantID INT NOT NULL, 
    ItemName VARCHAR(100) NOT NULL, 
    Price DECIMAL(10, 2) NOT NULL, 
    Availability BOOLEAN DEFAULT TRUE, 
    FOREIGN KEY (RestaurantID) REFERENCES Restaurant(RestaurantID) ON DELETE CASCADE 
); 

-- Create Orders table  
CREATE TABLE Orders ( 
    OrderID INT PRIMARY KEY, 
    CustomerID INT NOT NULL, 
    RestaurantID INT NOT NULL, 
    DeliveryAgentID INT, 
    OrderDate TIMESTAMP NOT NULL, 
    DeliveryAddress TEXT NOT NULL, 
    Status VARCHAR(20) NOT NULL CHECK (Status IN ('Placed', 'Preparing', 'Ready', 'On the way', 'Delivered', 'Cancelled')), 
    TotalAmount DECIMAL(10, 2) NOT NULL, 
    RewardCoinsEarned INT DEFAULT 0, 
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID) ON DELETE CASCADE, 
    FOREIGN KEY (RestaurantID) REFERENCES Restaurant(RestaurantID) ON DELETE CASCADE, 
    FOREIGN KEY (DeliveryAgentID) REFERENCES DeliveryAgent(DeliveryAgentID) ON DELETE SET NULL 
); 

-- Create OrderDetails table 
CREATE TABLE OrderDetails ( 
    OrderDetailsID INT PRIMARY KEY, 
    OrderID INT NOT NULL, 
    MenuID INT NOT NULL, 
    Quantity INT NOT NULL, 
    Price DECIMAL(10, 2) NOT NULL, 
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE, 
    FOREIGN KEY (MenuID) REFERENCES Menu(MenuID) ON DELETE CASCADE 
); 

-- Create Payment table 
CREATE TABLE Payment ( 
    TransactionID INT PRIMARY KEY, 
    OrderID INT NOT NULL, 
    PaymentMethod VARCHAR(50) NOT NULL, 
    TotalAmount DECIMAL(10, 2) NOT NULL, 
    RewardCoinsUsed INT DEFAULT 0, 
    AmountPaid DECIMAL(10, 2) NOT NULL, 
    PlatformCommission DECIMAL(10, 2) NOT NULL, 
    RestaurantEarnings DECIMAL(10, 2) NOT NULL, 
    PaymentStatus VARCHAR(20) NOT NULL CHECK (PaymentStatus IN ('Pending', 'Completed', 'Failed', 'Refunded')), 
    PaymentDate TIMESTAMP NOT NULL, 
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE 
); 

-- Create RewardCoinUsage table 
CREATE TABLE RewardCoinUsage ( 
    RewardCoinUsageID INT PRIMARY KEY, 
    TransactionID INT NOT NULL, 
    CustomerID INT NOT NULL, 
    CoinsUsed INT NOT NULL, 
    CoinsValue DECIMAL(10, 2) NOT NULL, 
    FOREIGN KEY (TransactionID) REFERENCES Payment(TransactionID) ON DELETE CASCADE, 
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID) ON DELETE CASCADE 
); 

-- Create PayoutHistory table 
CREATE TABLE PayoutHistory ( 
    PayoutID INT PRIMARY KEY, 
    DeliveryAgentID INT NOT NULL, 
    OrderID INT NOT NULL, 
    BaseFee DECIMAL(10, 2) NOT NULL, 
    DistanceFee DECIMAL(10, 2) NOT NULL, 
    TotalPayout DECIMAL(10, 2) NOT NULL, 
    PaymentStatus VARCHAR(20) NOT NULL CHECK (PaymentStatus IN ('Pending', 'Completed', 'Failed')), 
    PayoutDate TIMESTAMP NOT NULL, 
    FOREIGN KEY (DeliveryAgentID) REFERENCES DeliveryAgent(DeliveryAgentID) ON DELETE CASCADE, 
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE 
); 

-- Create FinancialReport table 
CREATE TABLE FinancialReport ( 
    ReportID INT PRIMARY KEY, 
    RestaurantID INT, 
    DeliveryAgentID INT, 
    CommissionEarned DECIMAL(10, 2) NOT NULL, 
    PayoutAmount DECIMAL(10, 2) NOT NULL, 
    ReportDate DATE NOT NULL, 
    FOREIGN KEY (RestaurantID) REFERENCES Restaurant(RestaurantID) ON DELETE SET NULL, 
    FOREIGN KEY (DeliveryAgentID) REFERENCES DeliveryAgent(DeliveryAgentID) ON DELETE SET NULL 
); 

-- Create Review table 
CREATE TABLE Review ( 
    ReviewID INT PRIMARY KEY, 
    OrderID INT NOT NULL, 
    Rating DECIMAL(2, 1) NOT NULL CHECK (Rating BETWEEN 0 AND 5), 
    Comment TEXT, 
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID) ON DELETE CASCADE 
); 

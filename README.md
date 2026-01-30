# Database Schema Design for E-Commerce App

This document outlines the database structure for your .NET & Flutter E-commerce graduation project.

## Overview
The system is built around **ASP.NET Core Identity** for user management. It includes core e-commerce features:
- **Catalog**: Categories & Products
- **Sales**: Orders, Order Items
- **Finance**: Payments

## 1. Identity & Users (ASP.NET Core Identity)
We will use the standard `AspNetCore.Identity` tables extended with custom fields.
- **AspNetUsers**:
  - `Id` (string, PK)
  - `FirstName` (string)
  - `LastName` (string)
  - `UserName`, `Email`, `PasswordHash`, `PhoneNumber`
- **AspNetRoles**: `Id`, `Name` (e.g. "Admin", "Customer")
- **AspNetUserRoles**: Mapping table between Users and Roles.
- **AspNetUserClaims**: Stats/Attributes for users.
- **AspNetUserLogins**: External auth providers (Google, Facebook).
- **AspNetUserTokens**: Logic tokens (Password Reset, Email Confirm, **Refresh Tokens**).
  - `UserId` (PK)
  - `LoginProvider` (PK)
  - `Name` (PK)
  - `Value` (The token string)

## 2. Catalog
- **Categories**:
  - `Id` (int)
  - `Name`
  - `ImageUrl`
  - `CreatedAt` (datetime)
- **Products**:
  - `Id` (int)
  - `Name`
  - `Description`
  - `Price` (decimal)
  - `StockQuantity` (int)
  - `CategoryId` (FK)
  - `CreatedAt` (datetime)
  - `UpdatedAt` (datetime)
- **ProductImages**:
  - `Id` (int)
  - `ProductId` (FK)
  - `ImageUrl`
  - `IsMain` (bool)

## 3. Social & Shopping Features (New)
- **ProductReviews**:
  - `Id` (int)
  - `ProductId` (FK)
  - `UserId` (FK)
  - `Rating` (int, 1-5)
  - `Comment` (string)
  - `CreatedAt` (datetime)
- **Wishlists**:
  - `Id` (int)
  - `UserId` (FK)
  - `ProductId` (FK)
  - `CreatedAt` (datetime)

## 4. Users & Identity
- **UserAddresses** (New - Users can save multiple addresses):
  - `Id` (int)
  - `UserId` (FK)
  - `FullAddress` (string, complete address for display)
  - `Street`
  - `City`
  - `State`
  - `ZipCode`
  - `Country`
  - `IsDefault` (bool)

## 5. Sales & Orders
- **Orders**:
  - `Id` (int)
  - `UserId` (FK)
  - `OrderDate`
  - `TotalAmount`
  - `ShippingCost` (decimal)
  - `ShippingAddress` (string, Snapshot of FullAddress)
  - `PaymentId` (FK, ID of the successful payment)
  - `ShipmentId` (FK, ID of the chosen shipment)
  - `CreatedAt`
- **OrderItems**:
  - `Id` (int)
  - `OrderId` (FK)
  - `ProductId` (FK)
  - `Quantity`
  - `UnitPrice`

## 7. Shipments (New)
- **Shipments**:
  - `Id` (int)
  - `OrderId` (FK)
  - `TrackingNumber` (string)
  - `Carrier` (string, e.g. "DHL", "Aramex")
  - `ShipmentDate` (datetime)
  - `EstimatedDelivery` (datetime)
  - `Cost` (decimal)

## 8. Paymob & Payments
- **Payments**:
  - `Id` (int)
  - `OrderId` (FK)
  - `PaymentMethod` (Enum: PaymentMethod)
  - `TransactionId`
  - `Amount`
  - `Status` (Enum: PaymentStatus)
  - `Status` (Enum: PaymentStatus)
  - `PaymentDate`
  - `Provider` (string, e.g. "Paymob", "PayPal")
  - `Details` (string, JSON/Generic field for MaskedPan, SubType, etc.)

## 9. Enums
- **OrderStatus**:
  - `Pending`, `Processing`, `Shipped`, `Delivered`, `Cancelled`, `Returned`
- **PaymentStatus**:
  - `Pending`, `Completed`, `Failed`, `Refunded`
- **PaymentMethod**:
  - `CreditCard`, `PayPal`, `COD` (Cash on Delivery)

## Visual Diagram (Mermaid)
You can copy the code below and insert it into Draw.io (Arrange > Insert > Advanced > Mermaid) or use it as a reference to draw manually.

```mermaid
classDiagram
    %% Enums
    class OrderStatus {
        <<enumeration>>
        Pending
        Processing
        Shipped
        Delivered
        Cancelled
        Returned
    }

    class PaymentStatus {
        <<enumeration>>
        Pending
        Completed
        Failed
        Refunded
    }

    class PaymentMethod {
        <<enumeration>>
        CreditCard
        PayPal
        COD
    }

    %% Identity 
    class AspNetUsers {
        string Id PK
        string FirstName
        string LastName
        string UserName
        string Email
        string PhoneNumber
        string PasswordHash
    }

    class AspNetRoles {
        string Id PK
        string Name
    }

    class AspNetUserRoles {
        string UserId FK
        string RoleId FK
    }

    class AspNetUserTokens {
        string UserId PK
        string LoginProvider PK
        string Name PK
        string Value
    }

    class UserAddress {
        int Id PK
        string UserId FK
        string FullAddress
        string Street
        string City
        string Country
        bool IsDefault
    }

    class AspNetRoles {
        string Id PK
        string Name
    }
    
    %% Catalog
    class Category {
        int Id PK
        string Name
        string ImageUrl
        datetime CreatedAt
    }
    
    class Product {
        int Id PK
        string Name
        string Description
        decimal Price
        int StockQuantity
        int CategoryId FK
        datetime CreatedAt
        datetime UpdatedAt
    }

    class ProductImage {
        int Id PK
        int ProductId FK
        string ImageUrl
        bool IsMain
    }

    %% Social
    class ProductReview {
        int Id PK
        int ProductId FK
        string UserId FK
        int Rating
        string Comment
        datetime CreatedAt
    }

    class Wishlist {
        int Id PK
        string UserId FK
        int ProductId FK
        datetime CreatedAt
    }

    %% Sales
    class Order {
        int Id PK
        string UserId FK
        datetime OrderDate
        decimal TotalAmount
        OrderStatus Status
        string ShippingAddress
        string TransactionId
        datetime CreatedAt
    }

    class OrderItem {
        int Id PK
        int OrderId FK
        int ProductId FK
        int Quantity
        decimal UnitPrice
    }

    %% Payments
    class Payment {
        int Id PK
        int OrderId FK
        PaymentMethod PaymentMethod
        string TransactionId
        decimal Amount
        PaymentStatus Status
        PaymentStatus Status
        datetime PaymentDate
        string ResponseCode
        string MaskedPan
        string SubType
    }

    %% Relationships
    Category "1" -- "*" Product : contains
    Product "1" -- "*" ProductImage : has_images
    Product "1" -- "*" ProductReview : has_reviews
    
    AspNetUsers "1" -- "*" AspNetUserTokens : has
    AspNetUsers "1" -- "*" AspNetUserRoles : owns
    AspNetRoles "1" -- "*" AspNetUserRoles : belongs_to
    
    AspNetUsers "1" -- "*" UserAddress : has_addresses
    AspNetUsers "1" -- "*" Order : places
    AspNetUsers "1" -- "*" ProductReview : writes
    AspNetUsers "1" -- "*" Wishlist : favorites

    Wishlist "*" -- "1" Product : contains

    Order "1" -- "*" OrderItem : contains
    Product "1" -- "*" OrderItem : included_in
    Order "1" -- "1" Payment : has_succeeded
    Order "1" -- "1" Shipment : has_chosen
```

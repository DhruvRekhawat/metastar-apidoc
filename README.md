# Metastar Artistverse Backend API Documentation

## Table of Contents

1. [API Overview](#api-overview)
2. [Authentication & Security](#authentication--security)
3. [Common Data Models](#common-data-models)
4. [User Authentication & Management](#user-authentication--management)
5. [User Profile & Address Management](#user-profile--address-management)
6. [Marketplace (General)](#marketplace-general)
7. [Merchandise (Cart & Orders)](#merchandise-cart--orders)
8. [Karaoke](#karaoke)
9. [Meet the Artist (Booking)](#meet-the-artist-booking)
10. [Meet the Artist (Request/Approval)](#meet-the-artist-requestapproval)
11. [Courses](#courses)
12. [Paid Videos](#paid-videos)
13. [Treasure Hunt](#treasure-hunt)
14. [Immersive Spaces (Popups/Videos)](#immersive-spaces-popupsvideos)
15. [Calendly Integration](#calendly-integration)
16. [Payment Gateway Webhooks](#payment-gateway-webhooks)
17. [Admin Panel](#admin-panel)
18. [New Dashboard](#new-dashboard)
19. [HTML Popup](#html-popup)
20. [Payment Gateway Callbacks (Internal Processing)](#payment-gateway-callbacks-internal-processing)

---

## API Overview

### Basic Information
- **Title**: Metastar Artistverse Backend API
- **Version**: v1.0
- **Description**: API documentation for Metastar Artistverse backend services
- **Base URL**: `http://localhost:3000` (development server)

### API Architecture
The API is structured around three main authentication contexts:

1. **User API** (`/api` paths) - Public user-facing endpoints
2. **Admin Panel** (`/admin` paths) - Administrative functions  
3. **New Dashboard** (`/dashboard` paths) - Modern dashboard interface

### Common Response Patterns

#### Standard Response Format
```json
{
  "status": boolean,
  "message": string,
  "data": object | null
}
```

#### Pagination Response Format
```json
{
  "status": boolean,
  "message": string,
  "result": {
    "items": array,
    "total": integer,
    "page": integer,
    "pageSize": integer,
    "totalPages": integer
  }
}
```

---

## Authentication & Security

### Authentication Schemes

#### 1. User Authentication (`userAuth`)
- **Type**: HTTP Bearer Token
- **Format**: JWT
- **Usage**: For `/api` endpoints
- **Header**: `Authorization: Bearer <token>`

#### 2. Admin Authentication (`adminAuth`)
- **Type**: HTTP Bearer Token
- **Format**: JWT
- **Usage**: For `/admin` endpoints
- **Header**: `Authorization: Bearer <token>`

#### 3. Dashboard Authentication (`dashboardAuth`)
- **Type**: HTTP Bearer Token
- **Format**: JWT
- **Usage**: For `/dashboard` endpoints
- **Header**: `Authorization: Bearer <token>`

### Middleware Components
- `verifyGoogleToken` - Validates Google OAuth tokens
- `verifyFirebaseToken` - Validates Firebase ID tokens
- `artistValidate` - Validates artist permissions
- `adminValidate` - Validates admin permissions
- `adminOnly` - Restricts access to admin users only

---

## Common Data Models

### UserIdentifierInput
```json
{
  "email": "string (email, nullable)",
  "mobile": "string (nullable)",
  "user_name": "string (nullable)"
}
```

### AddressDetails
```json
{
  "id": "integer (read-only)",
  "first_name": "string",
  "last_name": "string",
  "address_line_one": "string",
  "address_line_two": "string (nullable)",
  "address_line_three": "string (nullable)",
  "city": "string",
  "state": "string",
  "pincode": "string",
  "phone": "string",
  "email": "string (email)",
  "country": "string"
}
```

### UserProfileDetails
```json
{
  "name": "string (nullable)",
  "email": "string (email, nullable)",
  "phone": "string (nullable)",
  "image": "string (url, nullable)",
  "defaultaddress": "integer (nullable)"
}
```

### PaymentDetails
```json
{
  "id": "integer",
  "redirectUrl": "string",
  "transactionId": "string"
}
```

### PaginationResult
```json
{
  "items": "array",
  "total": "integer",
  "page": "integer",
  "pageSize": "integer",
  "totalPages": "integer"
}
```

---

## User Authentication & Management

### POST /api/check-user
**Purpose**: Checks if a user exists based on email, mobile, or username

**Request Body**:
```json
{
  "email": "string (optional)",
  "mobile": "string (optional)",
  "user_name": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Operation successful.",
  "exists": true
}
```

### POST /api/user
**Purpose**: Registers a new user and sends OTP for verification

**Request Body**:
```json
{
  "userName": "string (email or mobile)",
  "password": "string",
  "client_id": "integer (default: 2, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Registration successful.",
  "data": {
    "user_details": {
      "id": "integer",
      "name": "string",
      "email": "string"
    }
  }
}
```

### POST /api/login
**Purpose**: Authenticates user with email/mobile and password

**Request Body**:
```json
{
  "userName": "string (email or mobile)",
  "password": "string"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Login successful.",
  "data": {
    "token": "string (JWT)"
  }
}
```

### POST /api/google-user
**Purpose**: Handles Google OAuth login/registration
**Middleware**: `verifyGoogleToken`

**Request Body**:
```json
{
  "token": "string (Google ID token)",
  "client_id": "integer (default: 2, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Google login successful.",
  "data": {
    "user": {
      "id": "integer",
      "name": "string",
      "email": "string"
    },
    "token": "string (JWT)"
  }
}
```

### POST /api/facebook-user
**Purpose**: Handles Facebook login/registration via Firebase
**Middleware**: `verifyFirebaseToken`

**Request Body**:
```json
{
  "token": "string (Firebase ID token)",
  "client_id": "integer (default: 2, optional)"
}
```

### POST /api/twitter-user
**Purpose**: Handles Twitter login/registration via Firebase
**Middleware**: `verifyFirebaseToken`

**Request Body**:
```json
{
  "token": "string (Firebase ID token)",
  "client_id": "integer (default: 2, optional)"
}
```

### POST /api/firebase-user
**Purpose**: Handles generic Firebase login/registration
**Middleware**: `verifyFirebaseToken`

**Request Body**:
```json
{
  "token": "string (Firebase ID token)",
  "client_id": "integer (default: 2, optional)"
}
```

### POST /api/forgot-password
**Purpose**: Initiates password reset by sending OTP

**Request Body**:
```json
{
  "userName": "string (email or mobile)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "OTP sent successfully."
}
```

### POST /api/update-password
**Purpose**: Updates user's password after OTP verification

**Request Body**:
```json
{
  "userName": "string (email or mobile)",
  "otp": "string",
  "new_password": "string"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Password updated successfully.",
  "data": {
    "token": "string (JWT)"
  }
}
```

### POST /api/send-otp
**Purpose**: Sends OTP for verification to registered user

**Request Body**:
```json
{
  "userName": "string (email or mobile)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "OTP sent successfully."
}
```

### POST /api/verify-otp
**Purpose**: Verifies OTP and returns JWT upon successful verification

**Request Body**:
```json
{
  "userName": "string (email or mobile)",
  "otp": "string"
}
```

**Response**:
```json
{
  "status": true,
  "message": "OTP verified successfully.",
  "data": {
    "token": "string (JWT)"
  }
}
```

---

## User Profile & Address Management

### GET /api/address
**Purpose**: Retrieves user's default or most recent address
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Address retrieved successfully.",
  "data": {
    "address": {
      "id": "integer",
      "first_name": "string",
      "last_name": "string",
      "address_line_one": "string",
      "city": "string",
      "state": "string",
      "pincode": "string",
      "phone": "string",
      "email": "string",
      "country": "string"
    }
  }
}
```

### POST /api/address
**Purpose**: Adds a new address for the user
**Authentication**: Required (`userAuth`)

**Request Body**: `AddressDetails` object

**Response**:
```json
{
  "status": true,
  "message": "Address added successfully."
}
```

### GET /api/getAddressAll
**Purpose**: Retrieves all addresses saved by the user
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Addresses retrieved successfully.",
  "data": {
    "address": [
      {
        "id": "integer",
        "first_name": "string",
        "last_name": "string",
        "address_line_one": "string",
        "city": "string",
        "state": "string",
        "pincode": "string",
        "phone": "string",
        "email": "string",
        "country": "string"
      }
    ]
  }
}
```

### POST /api/updateAddress
**Purpose**: Updates an existing user address
**Authentication**: Required (`userAuth`)

**Request Body**: `AddressDetails` object with required `id` field

**Response**:
```json
{
  "status": true,
  "message": "Address updated successfully."
}
```

### GET /api/getUserProfile
**Purpose**: Retrieves the user's profile information
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Profile retrieved successfully.",
  "data": {
    "user": [
      {
        "name": "string",
        "email": "string",
        "phone": "string",
        "image": "string (url)",
        "defaultaddress": "integer"
      }
    ]
  }
}
```

### POST /api/insertUserProfile
**Purpose**: Inserts a new user profile entry
**Authentication**: Required (`userAuth`)

**Request Body**: `UserProfileDetails` object

**Response**:
```json
{
  "status": true,
  "message": "Profile inserted successfully."
}
```

### POST /api/updateUserProfile
**Purpose**: Updates an existing user profile
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "name": "string (optional)",
  "email": "string (optional)",
  "phone": "string (optional)",
  "default_address": "integer (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Profile updated successfully."
}
```

### POST /api/updateUserProfilePic
**Purpose**: Uploads and updates user's profile picture
**Authentication**: Required (`userAuth`)
**Content-Type**: `multipart/form-data`

**Request Body**:
```json
{
  "file": "binary (image file)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Profile picture updated successfully."
}
```

---

## Marketplace (General)

### GET /api/allProducts
**Purpose**: Retrieves a list of all active products (Merch, Courses, Meets, Paid Videos)
**Authentication**: Optional (`userAuth`)
**Note**: Includes wishlist status if authenticated

**Query Parameters**:
- `artistId` (string, optional): Comma-separated list of artist IDs
- `categoryId` (string, optional): Comma-separated list of category IDs
- `countryId` (string, optional): Comma-separated list of country IDs

**Response**:
```json
{
  "status": true,
  "message": "Products retrieved successfully.",
  "data": {
    "products": [
      {
        "id": "integer",
        "name": "string",
        "type": "string (MERCH|COURSE|MEET|PAID_VIDEO)",
        "wishlist_status": "boolean (nullable)"
      }
    ]
  }
}
```

### GET /api/allCategories
**Purpose**: Retrieves a list of all active product categories

**Response**:
```json
{
  "status": true,
  "message": "Categories retrieved successfully.",
  "data": [
    {
      "id": "integer",
      "name": "string",
      "description": "string"
    }
  ]
}
```

### POST /api/addWishlist
**Purpose**: Adds an item to user's wishlist
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "id": "integer",
  "type": "string (PRODUCT|MEET|COURSE|PAID_VIDEO)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Item added to wishlist successfully."
}
```

### POST /api/removeWishlist
**Purpose**: Removes an item from user's wishlist
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "id": "integer",
  "type": "string (PRODUCT|MEET|COURSE|PAID_VIDEO)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Item removed from wishlist successfully."
}
```

### GET /api/payment-gateways
**Purpose**: Retrieves available payment gateways
**Authentication**: Required (`userAuth`)

**Query Parameters**:
- `currency` (integer, optional): Currency ID (e.g., 1 for INR, 2 for USD)

**Response**:
```json
{
  "status": true,
  "message": "Payment gateways retrieved successfully.",
  "data": [
    {
      "id": "integer",
      "name": "string",
      "type": "string",
      "is_active": "boolean"
    }
  ]
}
```

---

## Merchandise (Cart & Orders)

### GET /api/products/{artistId}
**Purpose**: Retrieves merchandise products for a specific artist

**Path Parameters**:
- `artistId` (integer, required): ID of the artist

**Response**:
```json
{
  "status": true,
  "message": "Products retrieved successfully.",
  "data": {
    "products": [
      {
        "id": "integer",
        "name": "string",
        "price": "number",
        "currency": "string",
        "images": ["string"]
      }
    ]
  }
}
```

### GET /api/product-details/{productId}
**Purpose**: Retrieves detailed information for a specific merchandise product

**Path Parameters**:
- `productId` (integer, required): ID of the product

**Response**:
```json
{
  "status": true,
  "message": "Product details retrieved successfully.",
  "data": {
    "imageList": [
      {
        "url": "string",
        "alt": "string"
      }
    ],
    "details": {
      "variants": [
        {
          "id": "integer",
          "name": "string",
          "price": "number",
          "stock": "integer"
        }
      ],
      "pricing": {
        "base_price": "number",
        "currency": "string"
      }
    }
  }
}
```

### POST /api/cart
**Purpose**: Adds or updates an item in the user's shopping cart
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "productItemId": "integer",
  "quantity": "integer (0 to remove)",
  "currency": "integer (default: 1, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Cart updated successfully."
}
```

### POST /api/bulk-cart
**Purpose**: Replaces the user's entire cart
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "products": [
    {
      "productItemId": "integer",
      "quantity": "integer",
      "currency": "integer"
    }
  ]
}
```

**Response**:
```json
{
  "status": true,
  "message": "Cart updated successfully."
}
```

### GET /api/cart/{artistId}
**Purpose**: Retrieves user's shopping cart items
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `artistId` (string, required): ID of the artist or 'undefined'

**Query Parameters**:
- `coupon` (string, optional): Coupon code to apply
- `currency` (integer, optional, default: 1): Currency ID

**Response**:
```json
{
  "status": true,
  "message": "Cart items retrieved successfully.",
  "data": {
    "cartItems": [
      {
        "productItemId": "integer",
        "quantity": "integer",
        "price": "number",
        "total": "number",
        "product": {
          "name": "string",
          "image": "string"
        }
      }
    ],
    "subtotal": "number",
    "tax": "number",
    "total": "number"
  }
}
```

### GET /api/cartSingle
**Purpose**: Calculates price for a single item with coupon
**Authentication**: Required (`userAuth`)

**Query Parameters**:
- `coupon` (string, optional): Coupon code
- `currency` (integer, optional): Currency ID
- `productItemId` (integer, required): ID of the item
- `type` (string, required): Type of item ('1': Product, '2': Meet, '3': Course, '4': Paid Video)

**Response**:
```json
{
  "status": true,
  "message": "Price calculated successfully.",
  "data": {
    "cartItems": [
      {
        "price": "number",
        "discount": "number",
        "total": "number"
      }
    ]
  }
}
```

### POST /api/order
**Purpose**: Places an order for cart items (PhonePe INR only)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "artistId": "integer (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "coupon": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Order placed successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (order_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/order-now
**Purpose**: Places an order for a single product item immediately
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "quantity": "integer",
  "productItemId": "integer",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "currency": "integer (default: 1, optional)",
  "payment_method": "integer (default: 2 PayPal, optional)",
  "coupon": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Order placed successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (order_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/v2/order
**Purpose**: Places an order for cart items (v2 - multiple payment methods)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "artistId": "integer (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "payment_method": "integer (default: 2 PayPal, optional)",
  "coupon": "string (optional)",
  "currency": "integer (default: 1, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Order placed successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (order_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/cancel-payment
**Purpose**: Marks a pending payment as declined/failed
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "mTxnId": "string (Merchant Transaction ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Payment cancelled successfully."
}
```

### GET /api/orders
**Purpose**: Retrieves user's orders (paginated)
**Authentication**: Required (`userAuth`)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 40): Items per page

**Response**:
```json
{
  "status": true,
  "message": "Orders retrieved successfully.",
  "data": {
    "orders": [
      {
        "order_id": "integer",
        "order_date": "string (date-time)",
        "total_amount": "number",
        "status": "string"
      }
    ],
    "totalPages": "integer"
  }
}
```

### GET /api/v1/orders
**Purpose**: Retrieves user's orders (paginated, v1)
**Authentication**: Required (`userAuth`)

**Query Parameters**:
- `page` (integer, default: 40): Page number
- `pageSize` (integer, default: 40): Items per page

**Response**: Same as `/api/orders`

### GET /api/order-detail
**Purpose**: Retrieves detailed information for a specific order item
**Authentication**: Required (`userAuth`)

**Query Parameters**:
- `order_item_id` (integer, required): ID of the order item
- `order_type` (string, required): Type of order ('PRODUCT', 'COURSE', 'MEET', 'PAID_VIDEO')

**Response**:
```json
{
  "status": true,
  "message": "Order details retrieved successfully.",
  "data": {
    "detail": {
      "order_id": "integer",
      "status": "string",
      "created_at": "string",
      "total_amount": "number"
    },
    "products": [
      {
        "name": "string",
        "quantity": "integer",
        "price": "number"
      }
    ]
  }
}
```

### POST /api/cancelOrder
**Purpose**: Cancels a placed merchandise order
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "order_id": "integer",
  "order_type": "string (PRODUCT)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Order cancelled successfully."
}
```

---

## Karaoke

### GET /api/getKaraokeTracks/{artistId}
**Purpose**: Retrieves karaoke tracks for a specific artist
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `artistId` (integer, required): ID of the artist

**Response**:
```json
{
  "status": true,
  "message": "Karaoke tracks retrieved successfully.",
  "data": {
    "track": [
      {
        "id": "integer",
        "title": "string",
        "duration": "string",
        "audio_url": "string",
        "lyrics": "string"
      }
    ]
  }
}
```

### POST /api/process-karaoke
**Purpose**: Mixes user's vocal with karaoke track
**Authentication**: Required (`userAuth`)
**Content-Type**: `multipart/form-data`

**Request Body**:
```json
{
  "track": "integer (karaoke track ID)",
  "file": "binary (user's vocal recording)"
}
```

**Response**: Returns mixed audio file as binary stream (audio/mpeg)

---

## Meet the Artist (Booking)

### POST /api/register-user
**Purpose**: Registers a user for an artist meet and returns available slots
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "mobile": "string (optional)",
  "artistMeetId": "integer",
  "name": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Registration successful.",
  "data": {
    "price": "number",
    "currencies": [
      {
        "id": "integer",
        "name": "string",
        "symbol": "string"
      }
    ],
    "tax": "number",
    "name": "string",
    "image": "string (url)",
    "duration": "string",
    "artsit_name": "string",
    "user": {
      "id": "integer",
      "name": "string"
    },
    "slots": {
      "available_dates": ["string"],
      "available_times": ["string"]
    }
  }
}
```

### POST /api/init-meet
**Purpose**: Initiates payment for an artist meet slot (PhonePe INR only)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "slotDate": "string (date)",
  "slotTime": "string (time)",
  "formId": "string",
  "artistMeetId": "integer",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "coupon": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Meet payment initiated successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (meet_log_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/v2/init-meet
**Purpose**: Initiates payment for an artist meet (v2 - multiple payment methods)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "artistMeetId": "integer",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "coupon": "string (optional)",
  "currency": "integer (default: 1, optional)",
  "payment_method": "integer (default: 2 PayPal, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Meet payment initiated successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (meet_log_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/meet-callback
**Purpose**: Handles PhonePe payment callbacks for artist meets (Webhook)
**Note**: This is an internal endpoint for payment processing

**Request Body**: PhonePe callback data (specific format)

**Response**: Returns "OK" or "SUCCESS" text

### POST /api/cancel-meet
**Purpose**: Cancels a pending meet payment
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "mTxnId": "string (Merchant Transaction ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Meet payment cancelled successfully."
}
```

### GET /api/getArtistBookDetail/{id}
**Purpose**: Retrieves details for a specific artist meet (with login)
**Authentication**: Required (`userAuth`)
**Note**: Includes user's request status

**Path Parameters**:
- `id` (integer, required): ID of the artist meet detail

**Response**:
```json
{
  "status": true,
  "message": "Artist meet details retrieved successfully.",
  "data": [
    {
      "id": "integer",
      "title": "string",
      "description": "string",
      "price": "number",
      "duration": "string",
      "artist_name": "string",
      "image": "string (url)",
      "request_status": "string (nullable)"
    }
  ]
}
```

### GET /api/getArtistBookWithoutLogin/{id}
**Purpose**: Retrieves details for a specific artist meet (no login required)

**Path Parameters**:
- `id` (integer, required): ID of the artist meet detail

**Response**: Same structure as `/api/getArtistBookDetail/{id}` but without `request_status`

---

## Meet the Artist (Request/Approval)

### POST /api/asset-upload
**Purpose**: Uploads assets (video/audio) for a meet request
**Authentication**: Required (`userAuth`)
**Content-Type**: `multipart/form-data`

**Query Parameters**:
- `id` (integer, required): ID related to the asset (e.g., artist meet detail ID)

**Request Body**:
```json
{
  "file": "binary (asset file)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Asset uploaded successfully.",
  "url": "string (file URL)"
}
```

### POST /api/meet-request
**Purpose**: Submits a meet request application
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "data": {
    "name": "string",
    "phone": "string",
    "email": "string",
    "video_url": "string",
    "message": "string"
  },
  "id": "integer (artist meet detail ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Meet request submitted successfully."
}
```

### GET /api/meet-request
**Purpose**: Retrieves meet request details for artist review
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Response**: Returns HTML content for the accept/reject form

### GET /api/reject-request-detail
**Purpose**: Retrieves details for rejecting a meet request
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Response**: Returns HTML content for the rejection feedback form

### POST /api/reject-request
**Purpose**: Rejects a meet request (from feedback form)
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Request Body**:
```json
{
  "feedback": "string (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Request rejected successfully."
}
```

### GET /api/reject-request
**Purpose**: Rejects a meet request (GET - likely intended as POST)
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Response**:
```json
{
  "status": true,
  "message": "Request rejected successfully."
}
```

### POST /api/accept-request
**Purpose**: Accepts a meet request (from accept/reject form)
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Response**:
```json
{
  "status": true,
  "message": "Request accepted successfully."
}
```

### GET /api/accept-request
**Purpose**: Accepts a meet request (GET - likely intended as POST)
**Authentication**: Required (`userAuth`)
**Middleware**: `artistValidate`

**Query Parameters**:
- `token` (string, required): JWT token containing request ID
- `id` (string, required): Request ID

**Response**:
```json
{
  "status": true,
  "message": "Request accepted successfully."
}
```

---

## Courses

### GET /api/course
**Purpose**: Retrieves a list of all active courses
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Courses retrieved successfully.",
  "data": {
    "courses": [
      {
        "id": "integer",
        "title": "string",
        "description": "string",
        "price": "number",
        "thumbnail": "string (url)",
        "duration": "string",
        "instructor": "string"
      }
    ]
  }
}
```

### GET /api/user-course
**Purpose**: Retrieves courses purchased by the logged-in user
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "User courses retrieved successfully.",
  "data": {
    "courses": [
      {
        "id": "integer",
        "title": "string",
        "purchase_date": "string",
        "progress": "number",
        "is_completed": "boolean"
      }
    ]
  }
}
```

### GET /api/course-detail/{courseId}
**Purpose**: Retrieves detailed information for a specific course
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `courseId` (integer, required): ID of the course

**Query Parameters**:
- `coupon` (string, optional): Coupon code

**Response**:
```json
{
  "status": true,
  "message": "Course details retrieved successfully.",
  "data": {
    "id": "integer",
    "title": "string",
    "description": "string",
    "price": "number",
    "is_purchased": "boolean",
    "chapters": [
      {
        "id": "integer",
        "title": "string",
        "videos": [
          {
            "id": "integer",
            "title": "string",
            "duration": "string",
            "video_url": "string",
            "is_watched": "boolean"
          }
        ]
      }
    ]
  }
}
```

### POST /api/course-callback
**Purpose**: Handles PhonePe payment callbacks for course purchases (Webhook)
**Note**: This is an internal endpoint for payment processing

**Request Body**: PhonePe callback data (specific format)

**Response**: Returns "OK" text

### POST /api/course-purchase
**Purpose**: Initiates payment for a course purchase (PhonePe INR only)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "courseId": "integer",
  "coupon": "string (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Course purchase initiated successfully.",
  "data": {
    "id": "integer (purchase_log_id)",
    "payment": {
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/v2/course-purchase
**Purpose**: Initiates payment for a course purchase (v2 - multiple payment methods)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "courseId": "integer",
  "coupon": "string (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "payment_method": "integer (default: 2 PayPal, optional)",
  "currency": "integer (default: 1, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Course purchase initiated successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (purchase_log_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/cancel-course-purchase
**Purpose**: Cancels a pending course purchase payment
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "mTxnId": "string (Merchant Transaction ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Course purchase cancelled successfully."
}
```

### GET /api/getCourseVideo
**Purpose**: Streams a course video
**Note**: Requires a valid session token (JWT)

**Query Parameters**:
- `session` (string, required): JWT token containing video ID and session ID

**Response**: Returns course video stream (video/mp4)

---

## Paid Videos

### GET /api/paid-videos
**Purpose**: Retrieves a list of all active paid videos (films)
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Paid videos retrieved successfully.",
  "data": {
    "videos": [
      {
        "id": "integer",
        "title": "string",
        "description": "string",
        "price": "number",
        "thumbnail": "string (url)",
        "duration": "string",
        "genre": "string",
        "release_date": "string"
      }
    ]
  }
}
```

### GET /api/paid-video-details/{videoId}
**Purpose**: Retrieves detailed information for a specific paid video
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `videoId` (integer, required): ID of the video

**Query Parameters**:
- `coupon` (string, optional): Coupon code

**Response**:
```json
{
  "status": true,
  "message": "Video details retrieved successfully.",
  "data": {
    "id": "integer",
    "title": "string",
    "description": "string",
    "price": "number",
    "trailer_stream_url": "string (nullable)",
    "video_stream_url": "string (nullable)",
    "is_subscribed": "boolean",
    "duration": "string",
    "quality": "string",
    "subtitles": ["string"]
  }
}
```

### GET /api/user-subscribed-videos
**Purpose**: Retrieves paid videos subscribed by the user
**Authentication**: Required (`userAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Subscribed videos retrieved successfully.",
  "data": {
    "videos": [
      {
        "id": "integer",
        "title": "string",
        "subscription_date": "string",
        "watch_progress": "number",
        "is_completed": "boolean"
      }
    ]
  }
}
```

### POST /api/subscribe-video
**Purpose**: Initiates payment for a paid video subscription (PhonePe INR only)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "videoId": "integer",
  "coupon": "string (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Video subscription initiated successfully.",
  "data": {
    "id": "integer (entry_id)",
    "payment": {
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/v2/subscribe-video
**Purpose**: Initiates payment for a paid video subscription (v2 - multiple payment methods)
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "videoId": "integer",
  "coupon": "string (optional)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "payment_method": "integer (default: 2 PayPal, optional)",
  "currency": "integer (default: 1, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Video subscription initiated successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (entry_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/paid-video-callback
**Purpose**: Handles PhonePe payment callbacks for paid video subscriptions (Webhook)
**Note**: This is an internal endpoint for payment processing

**Request Body**: PhonePe callback data (specific format)

**Response**: Returns "OK" text

### POST /api/cancel-video-purchase
**Purpose**: Cancels a pending paid video purchase payment
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "mTxnId": "string (Merchant Transaction ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Video purchase cancelled successfully."
}
```

---

## Treasure Hunt

### GET /api/treasure/{huntId}/{coinId}
**Purpose**: Retrieves details for a specific treasure coin
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `huntId` (integer, required): ID of the treasure hunt
- `coinId` (integer, required): ID of the treasure coin

**Response**:
```json
{
  "status": true,
  "message": "Treasure coin details retrieved successfully.",
  "data": {
    "claimed": "boolean",
    "coins_collected": "integer",
    "productClaimed": "boolean",
    "hunt_details": {
      "id": "integer",
      "name": "string",
      "description": "string"
    },
    "coin_details": {
      "id": "integer",
      "value": "integer",
      "reward_type": "string"
    }
  }
}
```

### POST /api/treasure
**Purpose**: Claims a treasure coin for the user
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "huntId": "integer",
  "coinId": "integer"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Treasure coin claimed successfully.",
  "data": {
    "claimed": "boolean",
    "coins_collected": "integer",
    "productClaimed": "boolean",
    "reward": {
      "type": "string",
      "value": "integer"
    }
  }
}
```

### POST /api/treasure-claim
**Purpose**: Claims a physical product reward for a completed treasure hunt
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "productItemId": "integer",
  "productClaimId": "integer (treasure holding entry ID)",
  "address": "AddressDetails"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Product claim submitted successfully."
}
```

### GET /api/treasure-products/{artistId}
**Purpose**: Retrieves treasure hunt reward products for an artist
**Authentication**: Required (`userAuth`)

**Path Parameters**:
- `artistId` (integer, required): ID of the artist

**Response**:
```json
{
  "status": true,
  "message": "Treasure products retrieved successfully.",
  "data": {
    "products": [
      {
        "id": "integer",
        "name": "string",
        "description": "string",
        "coin_cost": "integer",
        "image": "string (url)",
        "stock": "integer"
      }
    ]
  }
}
```

---

## Immersive Spaces (Popups/Videos)

### GET /api/popup
**Purpose**: Retrieves content for a specific popup

**Query Parameters**:
- `id` (integer, required): ID of the popup

**Response**:
```json
{
  "status": true,
  "message": "Popup content retrieved successfully.",
  "data": {
    "title": "string",
    "images": ["string (image URLs)"],
    "text": "string",
    "audioLink": "string (url, nullable)",
    "video": "string (nullable)",
    "video_stream_url": "string (nullable)"
  }
}
```

### GET /api/video
**Purpose**: Retrieves details and stream URL for a specific video

**Query Parameters**:
- `id` (integer, required): ID of the video

**Response**:
```json
{
  "status": true,
  "message": "Video details retrieved successfully.",
  "data": {
    "video_id": "integer",
    "video": "string",
    "video_stream_url": "string",
    "title": "string",
    "description": "string",
    "duration": "string"
  }
}
```

---

## Calendly Integration

### POST /api/getAvailableSlot
**Purpose**: Fetches available time slots for a Calendly-integrated artist meet
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "artistMeetId": "integer"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Available slots retrieved successfully.",
  "data": {
    "available_times": [
      "2023-10-27T10:00:00",
      "2023-10-27T11:00:00"
    ],
    "calendar_url": "string",
    "timezone": "string"
  }
}
```

### POST /api/start-calendly-checkout
**Purpose**: Creates a pending checkout entry for a Calendly meet
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "artistMeetId": "integer",
  "coupon": "string (optional)",
  "email_id": "string (email)",
  "address": "AddressDetails (optional)",
  "address_id": "integer (optional)",
  "currency": "integer (default: 1, optional)",
  "payment_method": "integer (default: 2 PayPal, optional)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Calendly checkout created successfully."
}
```

### POST /api/start-calendly-payment
**Purpose**: Initiates payment for a Calendly meet checkout entry
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "event_id": "integer (Calendly checkout entry ID)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Calendly payment initiated successfully.",
  "data": {
    "selectedPayment": {
      "id": "integer (meet_log_id)",
      "redirectUrl": "string",
      "transactionId": "string"
    }
  }
}
```

### POST /api/cancel-calendly-event
**Purpose**: Cancels a Calendly event booking
**Authentication**: Required (`userAuth`)

**Request Body**:
```json
{
  "source_id": "string (Calendly event type source ID)",
  "email_id": "string (email)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Calendly event cancelled successfully."
}
```

---

## Payment Gateway Webhooks

### POST /api/paypal-callback
**Purpose**: Handles PayPal webhook notifications
**Note**: This is an internal endpoint for payment processing

**Request Body**: PayPal webhook data (specific format)

**Response**: Returns "SUCCESS" text

### POST /api/v2/pay-glocal-wh
**Purpose**: Handles PayGlocal webhook notifications
**Note**: This is an internal endpoint for payment processing

**Request Body**: PayGlocal webhook data (specific format)

**Response**: Returns "SUCCESS" text

### POST /api/v2/pay-wh
**Purpose**: Handles Razorpay webhook notifications
**Note**: This is an internal endpoint for payment processing

**Request Body**: Razorpay webhook data (specific format)

**Response**: Returns "SUCCESS" text

### POST /api/calendly-webhook
**Purpose**: Handles Calendly webhook notifications (invitee.created)
**Note**: This is an internal endpoint for payment processing

**Request Body**: Calendly webhook data (invitee.created event format)

**Response**: Returns "OK" text

---

## HTML Popup

### GET /api/getHTMLPopup
**Purpose**: Retrieves raw HTML content for a specific popup ID

**Query Parameters**:
- `popupId` (integer, required): ID of the HTML popup

**Response**:
```json
{
  "status": true,
  "message": "HTML popup retrieved successfully.",
  "data": {
    "htmlContent": "string (raw HTML)"
  }
}
```

---

## Payment Gateway Callbacks (Internal Processing)

### POST /api/pay-wh
**Purpose**: Handles PhonePe payment callbacks for merchandise orders
**Note**: This is an internal endpoint for payment processing

**Request Body**: PhonePe callback data for merchandise (specific format)

**Response**: Returns "SUCCESS" text

---

## Admin Panel

### POST /admin/login
**Purpose**: Logs in an admin user

**Request Body**:
```json
{
  "userName": "string (admin email or mobile)",
  "password": "string"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Admin login successful.",
  "data": {
    "token": "string (JWT)"
  }
}
```

### POST /admin/google-login
**Purpose**: Handles Google OAuth login for admin users
**Middleware**: `verifyGoogleToken`

**Request Body**:
```json
{
  "token": "string (Google ID token)"
}
```

**Response**:
```json
{
  "status": true,
  "message": "Admin Google login successful.",
  "data": {
    "token": "string (JWT)"
  }
}
```

### GET /admin/user
**Purpose**: Retrieves a paginated list of users for admin
**Authentication**: Required (`adminAuth`)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "user_id"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)

**Response**:
```json
{
  "status": true,
  "message": "Users retrieved successfully.",
  "result": {
    "items": [
      {
        "id": "integer",
        "name": "string",
        "email": "string",
        "created_at": "string",
        "last_login": "string"
      }
    ],
    "total": "integer"
  }
}
```

### GET /admin/karaoke
**Purpose**: Retrieves paginated karaoke recordings for admin
**Authentication**: Required (`adminAuth`)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "user_id"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artistId` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Karaoke recordings retrieved successfully.",
  "result": {
    "items": [
      {
        "id": "integer",
        "user_id": "integer",
        "artist_id": "integer",
        "track_name": "string",
        "created_at": "string",
        "file_url": "string"
      }
    ],
    "total": "integer"
  }
}
```

### GET /admin/orderHistory
**Purpose**: Retrieves paginated order history for admin
**Authentication**: Required (`adminAuth`)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "created"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artistId` (integer, optional): Filter by artist ID
- `filterBy` (string, optional): Status filter (e.g., "PLACED")

**Response**:
```json
{
  "status": true,
  "message": "Order history retrieved successfully.",
  "result": {
    "items": [
      {
        "order_id": "integer",
        "user_id": "integer",
        "total_amount": "number",
        "status": "string",
        "created_at": "string",
        "payment_method": "string"
      }
    ],
    "total": "integer"
  }
}
```

### GET /admin/cartHistory
**Purpose**: Retrieves paginated user shopping carts for admin
**Authentication**: Required (`adminAuth`)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "cart_id"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)

**Response**:
```json
{
  "status": true,
  "message": "Cart history retrieved successfully.",
  "result": {
    "items": [
      {
        "cart_id": "integer",
        "user_id": "integer",
        "total_items": "integer",
        "total_amount": "number",
        "created_at": "string"
      }
    ],
    "total": "integer"
  }
}
```

### GET /admin/orderDetail
**Purpose**: Retrieves order detail for admin
**⚠️ SECURITY RISK**: Missing `adminValidate` middleware

**Query Parameters**:
- `order_item_id` (integer, required): Order item ID
- `order_type` (string, required): Order type ('PRODUCT', 'COURSE', 'MEET', 'PAID_VIDEO')

**Response**:
```json
{
  "status": true,
  "message": "Order details retrieved successfully.",
  "data": {
    "detail": {
      "order_id": "integer",
      "status": "string",
      "created_at": "string",
      "total_amount": "number"
    },
    "products": [
      {
        "name": "string",
        "quantity": "integer",
        "price": "number"
      }
    ]
  }
}
```

### GET /admin/user-report
**Purpose**: Generates and downloads user Excel report
**⚠️ SECURITY RISK**: Missing `adminValidate` middleware

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `clientId` (integer, optional): Filter by client ID
- `loginType` (integer, optional): Filter by login type

**Response**: Returns Excel file download (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)

### GET /admin/karaoke-report
**Purpose**: Generates and downloads karaoke Excel report
**⚠️ SECURITY RISK**: Missing `adminValidate` middleware

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artistId` (integer, optional): Filter by artist ID

**Response**: Returns Excel file download (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)

### GET /admin/order-report
**Purpose**: Generates and downloads order Excel report
**⚠️ SECURITY RISK**: Missing `adminValidate` middleware

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artistId` (integer, optional): Filter by artist ID

**Response**: Returns Excel file download (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)

### GET /admin/user-filter
**Purpose**: Retrieves filter options for user reports/listings
**Authentication**: Required (`adminAuth`)

**Response**:
```json
{
  "status": true,
  "message": "Filter options retrieved successfully.",
  "data": {
    "loginTypes": [
      {
        "id": "integer",
        "name": "string"
      }
    ],
    "clients": [
      {
        "id": "integer",
        "name": "string"
      }
    ]
  }
}
```

---

## New Dashboard

### POST /dashboard/login
**Purpose**: Logs in a dashboard user (Admin or Artiste)

**Request Body**:
```json
{
  "email": "string (email)",
  "password": "string"
}
```

**Response**:
```json
{
  "message": "Login successful",
  "token": "string (JWT)",
  "expiresAt": "string (date-time)",
  "user": {
    "id": "integer",
    "name": "string",
    "email": "string (email)",
    "userType": "integer",
    "artistMap": "object"
  }
}
```

### POST /dashboard/logout
**Purpose**: Logs out the dashboard user
**Authentication**: Required (`dashboardAuth`)

**Response**:
```json
{
  "message": "Logout successful"
}
```

### GET /dashboard/getUsers
**Purpose**: Retrieves paginated list of users for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly` (role check)

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "user_id"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)
- `artist_id` (integer, optional): Filter by artist ID (intended but not implemented)

**Response**:
```json
{
  "status": true,
  "message": "Users retrieved successfully.",
  "result": {
    "items": [
      {
        "id": "integer",
        "name": "string",
        "email": "string",
        "created_at": "string"
      }
    ],
    "total": "integer",
    "page": "integer",
    "pageSize": "integer",
    "totalPages": "integer"
  }
}
```

### GET /dashboard/getKaraoke
**Purpose**: Retrieves paginated karaoke recordings for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "user_id"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Karaoke recordings retrieved successfully.",
  "result": {
    "items": [
      {
        "id": "integer",
        "user_id": "integer",
        "track_name": "string",
        "created_at": "string"
      }
    ],
    "total": "integer",
    "page": "integer",
    "pageSize": "integer",
    "totalPages": "integer"
  }
}
```

### GET /dashboard/getOrders
**Purpose**: Retrieves paginated sales items for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `key` (string, default: "created"): Sort key
- `order` (string, default: "desc"): Sort order (asc/desc)
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Orders retrieved successfully.",
  "result": {
    "items": [
      {
        "order_id": "integer",
        "total_amount": "number",
        "status": "string",
        "created_at": "string"
      }
    ],
    "total": "integer",
    "page": "integer",
    "pageSize": "integer",
    "totalPages": "integer"
  }
}
```

### GET /dashboard/getTopFans
**Purpose**: Retrieves list of top fans for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `timeline` (string, default: "weekly"): Time period (weekly/monthly/yearly)
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Top fans retrieved successfully.",
  "result": {
    "top_fans": [
      {
        "user_id": "integer",
        "user_name": "string",
        "purchase_amount": "number"
      }
    ]
  }
}
```

### GET /dashboard/getTopSongs
**Purpose**: Retrieves list of top karaoke songs for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artist_id` (integer, optional): Filter by artist ID
- `limit` (integer, default: 5): Number of results to return

**Response**:
```json
{
  "status": true,
  "message": "Top songs retrieved successfully.",
  "result": {
    "top_songs": [
      {
        "song_id": "integer",
        "song_title": "string",
        "play_count": "integer"
      }
    ]
  }
}
```

### GET /dashboard/getTimeSpent
**Purpose**: Retrieves average session duration from Google Analytics
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `startDate` (string, default: "30daysAgo"): Start date
- `endDate` (string, default: "today"): End date
- `compareStartDate` (string, default: "60daysAgo"): Comparison start date
- `compareEndDate` (string, default: "31daysAgo"): Comparison end date
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Time spent data retrieved successfully.",
  "result": {
    "average": "number (nullable)",
    "percentChange": "number (nullable)",
    "trend": [
      {
        "date": "string",
        "value": "number"
      }
    ],
    "isPositive": "boolean (nullable)"
  }
}
```

### GET /dashboard/getVisitsData
**Purpose**: Retrieves session count data from Google Analytics
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `startDate` (string, default: "30daysAgo"): Start date
- `endDate` (string, default: "today"): End date
- `compareStartDate` (string, default: "60daysAgo"): Comparison start date
- `compareEndDate` (string, default: "31daysAgo"): Comparison end date
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Visits data retrieved successfully.",
  "result": {
    "total": "number (nullable)",
    "percentChange": "number (nullable)",
    "trend": [
      {
        "date": "string",
        "value": "number"
      }
    ],
    "isPositive": "boolean (nullable)"
  }
}
```

### GET /dashboard/getCountriesData
**Purpose**: Retrieves session and revenue data by country from Google Analytics
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `startDate` (string, default: "30daysAgo"): Start date
- `endDate` (string, default: "today"): End date
- `page` (integer, default: 1): Page number
- `pageSize` (integer, default: 10): Items per page
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Countries data retrieved successfully.",
  "result": {
    "items": [
      {
        "country": "string",
        "sessions": "integer",
        "revenue": "number"
      }
    ],
    "total": "integer",
    "page": "integer",
    "pageSize": "integer",
    "totalPages": "integer"
  }
}
```

### GET /dashboard/getGenderData
**Purpose**: Retrieves user gender distribution from Google Analytics
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `startDate` (string, default: "30daysAgo"): Start date
- `endDate` (string, default: "today"): End date
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Gender data retrieved successfully.",
  "result": {
    "male": {
      "value": "integer",
      "percentage": "number"
    },
    "female": {
      "value": "integer",
      "percentage": "number"
    },
    "other": {
      "value": "integer",
      "percentage": "number"
    },
    "total": "integer"
  }
}
```

### GET /dashboard/getDeviceData
**Purpose**: Retrieves user device and browser data from Google Analytics
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `startDate` (string, default: "30daysAgo"): Start date
- `endDate` (string, default: "today"): End date
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Device data retrieved successfully.",
  "result": {
    "devices": [
      {
        "name": "string",
        "value": "integer"
      }
    ],
    "browsers": [
      {
        "name": "string",
        "value": "integer"
      }
    ],
    "total": "integer"
  }
}
```

### GET /dashboard/getTotalRevenue
**Purpose**: Retrieves total revenue and order count data
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `timeline` (string, default: "weekly"): Time period (weekly/monthly/yearly)
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Total revenue data retrieved successfully.",
  "result": {
    "total_revenue": "number",
    "percent_change": "string",
    "chart_data": [
      {
        "period": "string",
        "revenue": "number"
      }
    ],
    "total_orders": "integer"
  }
}
```

### GET /dashboard/getTrendingProducts
**Purpose**: Retrieves list of top-selling products for dashboard
**Authentication**: Required (`dashboardAuth`)
**Middleware**: `adminOnly`

**Query Parameters**:
- `timeline` (string, default: "weekly"): Time period (weekly/monthly/yearly)
- `artist_id` (integer, optional): Filter by artist ID

**Response**:
```json
{
  "status": true,
  "message": "Trending products retrieved successfully.",
  "result": {
    "trending_products": [
      {
        "product_id": "integer",
        "product_name": "string",
        "product_type": "string",
        "items_sold": "integer"
      }
    ]
  }
}
```

### GET /dashboard/user-report
**Purpose**: Generates and downloads user Excel report for dashboard
**⚠️ SECURITY RISK**: Missing authentication

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `clientId` (integer, optional): Filter by client ID
- `loginType` (integer, optional): Filter by login type

**Response**: Returns Excel file download (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)

### GET /dashboard/karaoke-report
**Purpose**: Generates and downloads karaoke Excel report for dashboard
**⚠️ SECURITY RISK**: Missing authentication

**Query Parameters**:
- `fromDate` (string, optional): Start date (YYYY-MM-DD)
- `toDate` (string, optional): End date (YYYY-MM-DD)
- `artistId` (integer, optional): Filter by artist ID

**Response**: Returns Excel file download (application/vnd.openxmlformats-officedocument.spreadsheetml.sheet)

---

## Security Notes

### ⚠️ Security Issues Identified

1. **Missing Authentication on Report Endpoints**:
   - `/admin/orderDetail`
   - `/admin/user-report`
   - `/admin/karaoke-report`
   - `/admin/order-report`
   - `/dashboard/user-report`
   - `/dashboard/karaoke-report`

2. **Duplicate GET/POST Routes**:
   - `/api/reject-request` (both GET and POST)
   - `/api/accept-request` (both GET and POST)

### Recommendations

1. Add proper authentication middleware to all report endpoints
2. Remove duplicate GET routes for request approval endpoints
3. Implement rate limiting for public endpoints
4. Add input validation for all endpoints
5. Implement proper CORS configuration
6. Add API versioning strategy
7. Implement comprehensive logging and monitoring

---

## Error Codes and Handling

### Common HTTP Status Codes
- **200**: Success (even for errors, check `status` field in response)
- **400**: Bad Request (invalid parameters)
- **401**: Unauthorized (invalid or missing authentication)
- **403**: Forbidden (insufficient permissions)
- **404**: Not Found (resource doesn't exist)
- **500**: Internal Server Error

### Error Response Format
```json
{
  "status": false,
  "message": "Error description",
  "data": null
}
```

---

## Rate Limiting and Best Practices

### Recommended Rate Limits
- **Authentication endpoints**: 5 requests per minute per IP
- **General API endpoints**: 100 requests per minute per user
- **File upload endpoints**: 10 requests per minute per user
- **Report generation**: 5 requests per hour per user

### Best Practices
1. Always include proper authentication headers
2. Validate all input parameters
3. Use pagination for list endpoints
4. Implement proper error handling
5. Cache frequently accessed data
6. Use HTTPS in production
7. Monitor API usage and performance
8. Implement proper logging

---

*End of API Documentation*

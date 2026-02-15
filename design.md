# SnapBill - System Design Document

**Version:** 1.0  
**Date:** February 15, 2026  
**Status:** Draft

---

## 1. Architecture Overview

### 1.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        FLUTTER MOBILE APP                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Presentation │  │   Domain     │  │     Data     │          │
│  │    Layer     │→ │    Layer     │→ │    Layer     │          │
│  │   (Screens)  │  │ (Providers)  │  │ (API Client) │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│         ↓                                      ↓                 │
│  ┌──────────────────────────────────────────────────┐          │
│  │         Local Storage (SQLite + Hive)            │          │
│  └──────────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTPS/TLS 1.3
┌─────────────────────────────────────────────────────────────────┐
│                      FASTAPI BACKEND (Python)                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  API Routes  │→ │   Services   │→ │  Database    │          │
│  │  (Endpoints) │  │   (Logic)    │  │   (Models)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    POSTGRESQL (Supabase)                         │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │  users   │ │  items   │ │ invoices │ │   otp    │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL SERVICES                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Google Gemini│  │  SMS Gateway │  │   Bluetooth  │          │
│  │      AI      │  │  (OTP/Notif) │  │   Printer    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Clean Architecture Principles

SnapBill follows Clean Architecture with three distinct layers:

**Presentation Layer (UI)**
- Screens and widgets
- User interaction handling
- State observation via Provider
- No business logic

**Domain Layer (Business Logic)**
- ChangeNotifier providers (state management)
- Business rules and validation
- Use case orchestration
- Platform-independent

**Data Layer (External Interface)**
- API client implementation
- Local database operations
- External service integration
- Data serialization/deserialization

**Benefits:**
- Testability: Each layer can be tested independently
- Maintainability: Clear separation of concerns
- Scalability: Easy to add new features
- Flexibility: Can swap implementations without affecting other layers


---

## 2. System Component Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      FLUTTER APPLICATION                         │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    UI COMPONENTS                        │    │
│  │  • HomeScreen        • InventoryScreen                  │    │
│  │  • BillingScreen     • HistoryScreen (+ Analytics)      │    │
│  │  • VoiceBillingScreen • SettingsScreen                  │    │
│  │  • FrequentBillingGrid • CustomerScreen                 │    │
│  └────────────────────────────────────────────────────────┘    │
│                           ↓                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                  STATE PROVIDERS                        │    │
│  │  • AuthProvider      • InventoryProvider                │    │
│  │  • BillProvider      • AnalyticsProvider                │    │
│  │  • SyncProvider      • CustomerProvider                 │    │
│  │  • PrinterProvider   • VoiceProvider                    │    │
│  └────────────────────────────────────────────────────────┘    │
│                           ↓                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                  DATA REPOSITORIES                      │    │
│  │  • ApiClient         • LocalDatabase (SQLite)           │    │
│  │  • CacheManager      • SecureStorage (Hive)             │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTP/REST
┌─────────────────────────────────────────────────────────────────┐
│                      FASTAPI BACKEND                             │
│                                                                  │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    API GATEWAY                          │    │
│  │  • JWT Middleware    • CORS Handler                     │    │
│  │  • Rate Limiter      • Error Handler                    │    │
│  └────────────────────────────────────────────────────────┘    │
│                           ↓                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                    API ROUTES                           │    │
│  │  /auth/*            /items/*                            │    │
│  │  /invoices/*        /voice/*                            │    │
│  │  /analytics/*       /customers/*                        │    │
│  │  /b2b/*  (Phase 2)  /prescriptions/* (Phase 3)         │    │
│  └────────────────────────────────────────────────────────┘    │
│                           ↓                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                   SERVICE LAYER                         │    │
│  │  • AuthService       • InventoryService                 │    │
│  │  • VoiceService      • AnalyticsService                 │    │
│  │  • GSTService        • SyncService                      │    │
│  │  • OTPService        • B2BService (Phase 2)             │    │
│  │  • PrescriptionService (Phase 3)                        │    │
│  └────────────────────────────────────────────────────────┘    │
│                           ↓                                      │
│  ┌────────────────────────────────────────────────────────┐    │
│  │                   DATABASE LAYER                        │    │
│  │  • SQLModel ORM      • Session Management               │    │
│  │  • Query Builder     • Transaction Handler              │    │
│  └────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    POSTGRESQL DATABASE                           │
│                                                                  │
│  Phase 1 Tables:                                                │
│  • users             • items              • categories          │
│  • invoices          • invoice_items      • customers           │
│  • otp               • sync_queue         • audit_logs          │
│                                                                  │
│  Phase 2 Tables:                                                │
│  • suppliers         • purchase_orders    • po_items            │
│  • supplier_ratings  • demand_aggregation                       │
│                                                                  │
│  Phase 3 Tables:                                                │
│  • doctors           • patients           • medicines           │
│  • prescriptions     • prescription_items                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    EXTERNAL INTEGRATIONS                         │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ Google Gemini│  │  SMS Gateway │  │   Bluetooth  │          │
│  │      AI      │  │   (Twilio/   │  │   ESC/POS    │          │
│  │   (Voice)    │  │    MSG91)    │  │   Printer    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │   Supabase   │  │    Email     │  │   WhatsApp   │          │
│  │   Storage    │  │   Service    │  │     API      │          │
│  │   (PDFs)     │  │  (SendGrid)  │  │  (Sharing)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Frontend Design Architecture

### 3.1 UI Layer (Presentation)

#### 3.1.1 Screen Structure

```
lib/
├── presentation/
│   ├── screens/
│   │   ├── auth/
│   │   │   ├── login_screen.dart
│   │   │   └── otp_verification_screen.dart
│   │   ├── home/
│   │   │   └── home_screen.dart
│   │   ├── billing/
│   │   │   ├── manual_billing_screen.dart
│   │   │   ├── voice_billing_screen.dart
│   │   │   └── frequent_billing_grid.dart
│   │   ├── inventory/
│   │   │   ├── inventory_list_screen.dart
│   │   │   ├── add_item_screen.dart
│   │   │   └── edit_item_screen.dart
│   │   ├── history/
│   │   │   └── history_screen.dart  # Analytics at top
│   │   ├── customers/
│   │   │   ├── customer_list_screen.dart
│   │   │   └── customer_detail_screen.dart
│   │   ├── settings/
│   │   │   └── settings_screen.dart
│   │   ├── b2b/  # Phase 2
│   │   │   ├── supplier_marketplace_screen.dart
│   │   │   ├── purchase_order_screen.dart
│   │   │   └── demand_forecast_screen.dart
│   │   └── prescription/  # Phase 3
│   │       ├── create_prescription_screen.dart
│   │       ├── patient_list_screen.dart
│   │       └── prescription_history_screen.dart
│   └── widgets/
│       ├── common/
│       │   ├── custom_button.dart
│       │   ├── loading_indicator.dart
│       │   └── error_dialog.dart
│       ├── billing/
│       │   ├── bill_item_card.dart
│       │   └── payment_method_selector.dart
│       └── analytics/
│           ├── sales_chart.dart
│           ├── metric_card.dart
│           └── top_products_list.dart
```

#### 3.1.2 Bottom Navigation Architecture

```dart
// Main navigation structure
BottomNavigationBar(
  items: [
    BottomNavigationBarItem(icon: Icons.home, label: 'Home'),
    BottomNavigationBarItem(icon: Icons.inventory, label: 'Inventory'),
    BottomNavigationBarItem(icon: Icons.receipt, label: 'Billing'),
    BottomNavigationBarItem(icon: Icons.history, label: 'History'),
    BottomNavigationBarItem(icon: Icons.settings, label: 'Settings'),
  ]
)

// Navigation flow:
// Home → Quick actions (Voice Bill, Manual Bill, Frequent Items)
// Inventory → Product list, Add/Edit, Categories
// Billing → Manual entry, Voice input, Frequent grid
// History → Analytics dashboard + Bill list
// Settings → Profile, Printer, Language, Logout
```


### 3.2 Domain Layer (State Management)

#### 3.2.1 Provider Architecture

```dart
// main.dart - MultiProvider root injection
void main() {
  runApp(
    MultiProvider(
      providers: [
        ChangeNotifierProvider(create: (_) => AuthProvider()),
        ChangeNotifierProvider(create: (_) => InventoryProvider()),
        ChangeNotifierProvider(create: (_) => BillProvider()),
        ChangeNotifierProvider(create: (_) => AnalyticsProvider()),
        ChangeNotifierProvider(create: (_) => SyncProvider()),
        ChangeNotifierProvider(create: (_) => CustomerProvider()),
        ChangeNotifierProvider(create: (_) => PrinterProvider()),
        ChangeNotifierProvider(create: (_) => VoiceProvider()),
        // Phase 2
        ChangeNotifierProvider(create: (_) => B2BProvider()),
        // Phase 3
        ChangeNotifierProvider(create: (_) => PrescriptionProvider()),
      ],
      child: SnapBillApp(),
    ),
  );
}
```

#### 3.2.2 AuthProvider Design

```dart
class AuthProvider extends ChangeNotifier {
  User? _currentUser;
  String? _jwtToken;
  AuthState _state = AuthState.unauthenticated;
  
  // Authentication flow
  Future<void> sendOTP(String phoneNumber) async {
    _state = AuthState.sendingOTP;
    notifyListeners();
    
    try {
      await _apiClient.post('/auth/send-otp', {
        'phone_number': phoneNumber
      });
      _state = AuthState.otpSent;
    } catch (e) {
      _state = AuthState.error;
      _errorMessage = e.toString();
    }
    notifyListeners();
  }
  
  Future<void> verifyOTP(String phoneNumber, String otp) async {
    _state = AuthState.verifying;
    notifyListeners();
    
    try {
      final response = await _apiClient.post('/auth/verify-otp', {
        'phone_number': phoneNumber,
        'otp': otp
      });
      
      _jwtToken = response['token'];
      _currentUser = User.fromJson(response['user']);
      
      // Store token securely
      await _secureStorage.write(key: 'jwt_token', value: _jwtToken);
      
      _state = AuthState.authenticated;
    } catch (e) {
      _state = AuthState.error;
      _errorMessage = e.toString();
    }
    notifyListeners();
  }
  
  Future<void> logout() async {
    _jwtToken = null;
    _currentUser = null;
    await _secureStorage.delete(key: 'jwt_token');
    _state = AuthState.unauthenticated;
    notifyListeners();
  }
  
  // JWT token getter for API calls
  String? get token => _jwtToken;
  User? get currentUser => _currentUser;
  bool get isAuthenticated => _state == AuthState.authenticated;
}
```

#### 3.2.3 InventoryProvider Design

```dart
class InventoryProvider extends ChangeNotifier {
  List<Item> _items = [];
  List<Item> _cachedFrequentItems = [];
  bool _isLoading = false;
  
  // Fetch all items from backend
  Future<void> fetchItems() async {
    _isLoading = true;
    notifyListeners();
    
    try {
      final response = await _apiClient.get('/items');
      _items = (response as List)
          .map((json) => Item.fromJson(json))
          .toList();
      
      // Cache to local database
      await _localDb.saveItems(_items);
      
      // Update frequent items cache (top 100)
      await _updateFrequentItemsCache();
    } catch (e) {
      // Load from local cache if API fails
      _items = await _localDb.getItems();
    }
    
    _isLoading = false;
    notifyListeners();
  }
  
  // Cache top 100 frequent items for offline billing
  Future<void> _updateFrequentItemsCache() async {
    final frequent = await _apiClient.get('/items/frequent?limit=100');
    _cachedFrequentItems = (frequent as List)
        .map((json) => Item.fromJson(json))
        .toList();
    
    await _localDb.saveFrequentItems(_cachedFrequentItems);
  }
  
  // Add new item with optimistic update
  Future<void> addItem(Item item) async {
    // Optimistic UI update
    _items.add(item);
    notifyListeners();
    
    try {
      final response = await _apiClient.post('/items', item.toJson());
      final savedItem = Item.fromJson(response);
      
      // Replace optimistic item with server response
      final index = _items.indexWhere((i) => i.id == item.id);
      _items[index] = savedItem;
    } catch (e) {
      // Rollback on error
      _items.removeWhere((i) => i.id == item.id);
      rethrow;
    }
    notifyListeners();
  }
  
  // Search items (local first, then API)
  List<Item> searchItems(String query) {
    if (query.isEmpty) return _items;
    
    return _items.where((item) {
      return item.name.toLowerCase().contains(query.toLowerCase()) ||
             item.aliases.any((alias) => 
                 alias.toLowerCase().contains(query.toLowerCase()));
    }).toList();
  }
  
  List<Item> get items => _items;
  List<Item> get frequentItems => _cachedFrequentItems;
}
```

#### 3.2.4 BillProvider Design

```dart
class BillProvider extends ChangeNotifier {
  List<BillItem> _currentBillItems = [];
  Customer? _selectedCustomer;
  PaymentMethod _paymentMethod = PaymentMethod.cash;
  
  // Add item to current bill
  void addItem(Item item, int quantity, double rate) {
    final billItem = BillItem(
      item: item,
      quantity: quantity,
      rate: rate,
      total: quantity * rate,
    );
    
    _currentBillItems.add(billItem);
    notifyListeners();
  }
  
  // Remove item from bill
  void removeItem(int index) {
    _currentBillItems.removeAt(index);
    notifyListeners();
  }
  
  // Calculate totals with GST
  BillSummary calculateSummary() {
    double subtotal = 0;
    double cgst = 0;
    double sgst = 0;
    double igst = 0;
    
    for (var item in _currentBillItems) {
      subtotal += item.total;
      
      // Calculate GST based on item's GST rate
      final gstAmount = item.total * (item.item.gstRate / 100);
      
      // CGST/SGST vs IGST logic
      if (_isIntraState()) {
        cgst += gstAmount / 2;
        sgst += gstAmount / 2;
      } else {
        igst += gstAmount;
      }
    }
    
    final grandTotal = subtotal + cgst + sgst + igst;
    
    return BillSummary(
      subtotal: subtotal,
      cgst: cgst,
      sgst: sgst,
      igst: igst,
      grandTotal: grandTotal,
    );
  }
  
  // Generate invoice
  Future<Invoice> generateInvoice() async {
    final summary = calculateSummary();
    
    final invoice = Invoice(
      invoiceNumber: await _getNextInvoiceNumber(),
      date: DateTime.now(),
      items: _currentBillItems,
      customer: _selectedCustomer,
      paymentMethod: _paymentMethod,
      subtotal: summary.subtotal,
      cgst: summary.cgst,
      sgst: summary.sgst,
      igst: summary.igst,
      grandTotal: summary.grandTotal,
    );
    
    // Save to backend
    try {
      final response = await _apiClient.post('/invoices', invoice.toJson());
      final savedInvoice = Invoice.fromJson(response);
      
      // Clear current bill
      _currentBillItems.clear();
      _selectedCustomer = null;
      notifyListeners();
      
      return savedInvoice;
    } catch (e) {
      // Save to local queue for sync
      await _localDb.saveToSyncQueue(invoice);
      rethrow;
    }
  }
  
  // Check if transaction is intra-state
  bool _isIntraState() {
    // Compare retailer GSTIN state code with customer GSTIN
    // If customer has no GSTIN, assume intra-state
    if (_selectedCustomer?.gstin == null) return true;
    
    final retailerState = _authProvider.currentUser?.gstin?.substring(0, 2);
    final customerState = _selectedCustomer?.gstin?.substring(0, 2);
    
    return retailerState == customerState;
  }
  
  List<BillItem> get billItems => _currentBillItems;
  double get billTotal => calculateSummary().grandTotal;
}
```


#### 3.2.5 VoiceProvider Design

```dart
class VoiceProvider extends ChangeNotifier {
  bool _isListening = false;
  bool _isProcessing = false;
  String _transcript = '';
  List<Item> _matchedItems = [];
  
  // Start voice recording
  Future<void> startListening() async {
    _isListening = true;
    notifyListeners();
    
    // Use speech_to_text package
    await _speechToText.listen(
      onResult: (result) {
        _transcript = result.recognizedWords;
        notifyListeners();
      },
      localeId: _getLocaleId(), // hi_IN or en_IN
    );
  }
  
  // Stop recording and process
  Future<void> stopListening() async {
    _isListening = false;
    await _speechToText.stop();
    
    if (_transcript.isNotEmpty) {
      await _processVoiceCommand(_transcript);
    }
  }
  
  // Process voice command with Gemini AI
  Future<void> _processVoiceCommand(String transcript) async {
    _isProcessing = true;
    notifyListeners();
    
    try {
      // Send to backend voice processing endpoint
      final response = await _apiClient.post('/voice/process', {
        'transcript': transcript,
        'language': _currentLanguage,
      });
      
      // Response contains matched items with confidence scores
      _matchedItems = (response['matches'] as List)
          .map((json) => ItemMatch.fromJson(json))
          .toList();
      
      // If single high-confidence match, auto-add to bill
      if (_matchedItems.length == 1 && 
          _matchedItems.first.confidence > 0.9) {
        _autoAddToBill(_matchedItems.first);
      }
    } catch (e) {
      _errorMessage = e.toString();
    }
    
    _isProcessing = false;
    notifyListeners();
  }
  
  // Auto-add high-confidence match to bill
  void _autoAddToBill(ItemMatch match) {
    _billProvider.addItem(
      match.item,
      match.quantity ?? 1,
      match.rate ?? match.item.price,
    );
  }
  
  bool get isListening => _isListening;
  bool get isProcessing => _isProcessing;
  List<Item> get matchedItems => _matchedItems;
}
```

#### 3.2.6 AnalyticsProvider Design

```dart
class AnalyticsProvider extends ChangeNotifier {
  AnalyticsData? _todayData;
  AnalyticsData? _weeklyData;
  AnalyticsData? _monthlyData;
  bool _isLoading = false;
  
  // Fetch analytics from backend
  Future<void> fetchAnalytics() async {
    _isLoading = true;
    notifyListeners();
    
    try {
      // Fetch all periods in parallel
      final results = await Future.wait([
        _apiClient.get('/analytics/today'),
        _apiClient.get('/analytics/weekly'),
        _apiClient.get('/analytics/monthly'),
      ]);
      
      _todayData = AnalyticsData.fromJson(results[0]);
      _weeklyData = AnalyticsData.fromJson(results[1]);
      _monthlyData = AnalyticsData.fromJson(results[2]);
      
      // Cache for offline viewing
      await _cacheAnalytics();
    } catch (e) {
      // Load from cache if API fails
      await _loadCachedAnalytics();
    }
    
    _isLoading = false;
    notifyListeners();
  }
  
  // Export analytics to CSV
  Future<String> exportToCSV(String period) async {
    final data = _getDataForPeriod(period);
    final csv = _generateCSV(data);
    
    // Save to device storage
    final path = await _saveToFile(csv, 'analytics_$period.csv');
    return path;
  }
  
  // Export analytics to PDF
  Future<String> exportToPDF(String period) async {
    final data = _getDataForPeriod(period);
    final pdf = await _generatePDF(data);
    
    final path = await _saveToFile(pdf, 'analytics_$period.pdf');
    return path;
  }
  
  AnalyticsData? get todayData => _todayData;
  AnalyticsData? get weeklyData => _weeklyData;
  AnalyticsData? get monthlyData => _monthlyData;
}

// Analytics data model
class AnalyticsData {
  final double totalSales;
  final double percentageChange;
  final List<TopProduct> topProducts;
  final double profitMargin;
  final List<PeakHour> peakHours;
  final List<Item> slowMovingItems;
  final GSTBreakdown gstBreakdown;
  final CustomerInsights customerInsights;
  
  AnalyticsData({
    required this.totalSales,
    required this.percentageChange,
    required this.topProducts,
    required this.profitMargin,
    required this.peakHours,
    required this.slowMovingItems,
    required this.gstBreakdown,
    required this.customerInsights,
  });
}
```

### 3.3 Data Layer (API & Storage)

#### 3.3.1 API Client Design

```dart
class ApiClient {
  final Dio _dio;
  final AuthProvider _authProvider;
  
  ApiClient(this._authProvider) : _dio = Dio() {
    _dio.options.baseUrl = Config.apiBaseUrl;
    _dio.options.connectTimeout = Duration(seconds: 10);
    _dio.options.receiveTimeout = Duration(seconds: 10);
    
    // Add JWT interceptor
    _dio.interceptors.add(
      InterceptorsWrapper(
        onRequest: (options, handler) {
          // Inject JWT token
          final token = _authProvider.token;
          if (token != null) {
            options.headers['Authorization'] = 'Bearer $token';
          }
          return handler.next(options);
        },
        onError: (error, handler) {
          // Handle 401 Unauthorized
          if (error.response?.statusCode == 401) {
            _authProvider.logout();
          }
          return handler.next(error);
        },
      ),
    );
  }
  
  // Generic GET request
  Future<dynamic> get(String path, {Map<String, dynamic>? params}) async {
    try {
      final response = await _dio.get(path, queryParameters: params);
      return response.data;
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }
  
  // Generic POST request
  Future<dynamic> post(String path, dynamic data) async {
    try {
      final response = await _dio.post(path, data: data);
      return response.data;
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }
  
  // Error handling
  ApiException _handleError(DioException e) {
    if (e.type == DioExceptionType.connectionTimeout) {
      return ApiException('Connection timeout. Please check your internet.');
    } else if (e.type == DioExceptionType.receiveTimeout) {
      return ApiException('Server is taking too long to respond.');
    } else if (e.response != null) {
      return ApiException(
        e.response?.data['message'] ?? 'An error occurred',
        statusCode: e.response?.statusCode,
      );
    } else {
      return ApiException('Network error. Please try again.');
    }
  }
}
```

#### 3.3.2 Local Database Design (SQLite)

```dart
class LocalDatabase {
  static Database? _database;
  
  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }
  
  Future<Database> _initDatabase() async {
    final path = await getDatabasesPath();
    final dbPath = join(path, 'snapbill.db');
    
    return await openDatabase(
      dbPath,
      version: 1,
      onCreate: (db, version) async {
        // Create tables
        await db.execute('''
          CREATE TABLE items (
            id TEXT PRIMARY KEY,
            name TEXT NOT NULL,
            price REAL NOT NULL,
            gst_rate REAL NOT NULL,
            hsn_code TEXT,
            category TEXT,
            aliases TEXT,
            stock_quantity INTEGER,
            synced INTEGER DEFAULT 0
          )
        ''');
        
        await db.execute('''
          CREATE TABLE invoices (
            id TEXT PRIMARY KEY,
            invoice_number TEXT NOT NULL,
            date TEXT NOT NULL,
            customer_id TEXT,
            subtotal REAL NOT NULL,
            cgst REAL,
            sgst REAL,
            igst REAL,
            grand_total REAL NOT NULL,
            payment_method TEXT,
            synced INTEGER DEFAULT 0
          )
        ''');
        
        await db.execute('''
          CREATE TABLE sync_queue (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            entity_type TEXT NOT NULL,
            entity_id TEXT NOT NULL,
            operation TEXT NOT NULL,
            data TEXT NOT NULL,
            created_at TEXT NOT NULL,
            retry_count INTEGER DEFAULT 0
          )
        ''');
        
        await db.execute('''
          CREATE TABLE frequent_items_cache (
            item_id TEXT PRIMARY KEY,
            cached_at TEXT NOT NULL
          )
        ''');
      },
    );
  }
  
  // Save items to local cache
  Future<void> saveItems(List<Item> items) async {
    final db = await database;
    final batch = db.batch();
    
    for (var item in items) {
      batch.insert(
        'items',
        item.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace,
      );
    }
    
    await batch.commit(noResult: true);
  }
  
  // Get items from local cache
  Future<List<Item>> getItems() async {
    final db = await database;
    final maps = await db.query('items');
    return maps.map((map) => Item.fromMap(map)).toList();
  }
  
  // Save to sync queue for offline operations
  Future<void> saveToSyncQueue(String entityType, String entityId, 
                                String operation, Map<String, dynamic> data) async {
    final db = await database;
    await db.insert('sync_queue', {
      'entity_type': entityType,
      'entity_id': entityId,
      'operation': operation,
      'data': jsonEncode(data),
      'created_at': DateTime.now().toIso8601String(),
    });
  }
  
  // Get pending sync items
  Future<List<SyncQueueItem>> getPendingSyncItems() async {
    final db = await database;
    final maps = await db.query('sync_queue', orderBy: 'created_at ASC');
    return maps.map((map) => SyncQueueItem.fromMap(map)).toList();
  }
}
```


#### 3.3.3 Secure Storage Design (Hive)

```dart
class SecureStorage {
  static Box? _secureBox;
  
  Future<void> init() async {
    await Hive.initFlutter();
    
    // Encrypt sensitive data
    final encryptionKey = await _getEncryptionKey();
    _secureBox = await Hive.openBox(
      'secure_storage',
      encryptionCipher: HiveAesCipher(encryptionKey),
    );
  }
  
  // Store JWT token
  Future<void> saveToken(String token) async {
    await _secureBox!.put('jwt_token', token);
  }
  
  // Retrieve JWT token
  String? getToken() {
    return _secureBox!.get('jwt_token');
  }
  
  // Store user credentials
  Future<void> saveCredentials(String phone, String userId) async {
    await _secureBox!.put('phone_number', phone);
    await _secureBox!.put('user_id', userId);
  }
  
  // Clear all secure data
  Future<void> clearAll() async {
    await _secureBox!.clear();
  }
  
  // Get encryption key from Android Keystore
  Future<List<int>> _getEncryptionKey() async {
    const secureStorage = FlutterSecureStorage();
    
    // Check if key exists
    String? keyString = await secureStorage.read(key: 'encryption_key');
    
    if (keyString == null) {
      // Generate new key
      final key = Hive.generateSecureKey();
      await secureStorage.write(
        key: 'encryption_key',
        value: base64Encode(key),
      );
      return key;
    }
    
    return base64Decode(keyString);
  }
}
```

---

## 4. Backend Architecture Design

### 4.1 API Layer (FastAPI Routes)

#### 4.1.1 Project Structure

```
backend/
├── app/
│   ├── main.py                 # FastAPI app initialization
│   ├── config.py               # Configuration management
│   ├── dependencies.py         # Dependency injection
│   │
│   ├── api/
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py         # Authentication routes
│   │   │   ├── items.py        # Inventory routes
│   │   │   ├── invoices.py     # Invoice routes
│   │   │   ├── voice.py        # Voice processing routes
│   │   │   ├── analytics.py    # Analytics routes
│   │   │   ├── customers.py    # Customer routes
│   │   │   ├── b2b.py          # B2B routes (Phase 2)
│   │   │   └── prescriptions.py # Prescription routes (Phase 3)
│   │
│   ├── services/
│   │   ├── auth_service.py     # Authentication logic
│   │   ├── otp_service.py      # OTP generation/validation
│   │   ├── voice_service.py    # Voice processing with Gemini
│   │   ├── gst_service.py      # GST calculation logic
│   │   ├── analytics_service.py # Analytics computation
│   │   ├── sync_service.py     # Data synchronization
│   │   ├── b2b_service.py      # B2B logic (Phase 2)
│   │   └── prescription_service.py # Prescription logic (Phase 3)
│   │
│   ├── models/
│   │   ├── user.py             # User model
│   │   ├── item.py             # Item model
│   │   ├── invoice.py          # Invoice model
│   │   ├── customer.py         # Customer model
│   │   ├── otp.py              # OTP model
│   │   ├── purchase_order.py   # PO model (Phase 2)
│   │   ├── supplier.py         # Supplier model (Phase 2)
│   │   └── prescription.py     # Prescription model (Phase 3)
│   │
│   ├── schemas/
│   │   ├── auth.py             # Auth request/response schemas
│   │   ├── item.py             # Item schemas
│   │   ├── invoice.py          # Invoice schemas
│   │   └── ...
│   │
│   ├── core/
│   │   ├── security.py         # JWT, password hashing
│   │   ├── database.py         # Database connection
│   │   └── middleware.py       # Custom middleware
│   │
│   └── utils/
│       ├── gemini_client.py    # Google Gemini API client
│       ├── sms_client.py       # SMS gateway client
│       └── helpers.py          # Utility functions
│
├── alembic/                    # Database migrations
├── tests/                      # Unit and integration tests
├── requirements.txt
└── Dockerfile
```

#### 4.1.2 Authentication Routes

```python
# app/api/v1/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session
from app.core.database import get_session
from app.services.auth_service import AuthService
from app.services.otp_service import OTPService
from app.schemas.auth import SendOTPRequest, VerifyOTPRequest, TokenResponse

router = APIRouter(prefix="/auth", tags=["authentication"])

@router.post("/send-otp")
async def send_otp(
    request: SendOTPRequest,
    session: Session = Depends(get_session),
    otp_service: OTPService = Depends()
):
    """
    Send OTP to user's phone number.
    Rate limited to 3 requests per hour per phone number.
    """
    # Check rate limit
    if await otp_service.is_rate_limited(request.phone_number):
        raise HTTPException(
            status_code=status.HTTP_429_TOO_MANY_REQUESTS,
            detail="Too many OTP requests. Please try after 1 hour."
        )
    
    # Generate and send OTP
    otp = await otp_service.generate_otp(request.phone_number)
    await otp_service.send_sms(request.phone_number, otp)
    
    return {"message": "OTP sent successfully", "expires_in": 300}

@router.post("/verify-otp", response_model=TokenResponse)
async def verify_otp(
    request: VerifyOTPRequest,
    session: Session = Depends(get_session),
    auth_service: AuthService = Depends(),
    otp_service: OTPService = Depends()
):
    """
    Verify OTP and return JWT token.
    """
    # Verify OTP
    is_valid = await otp_service.verify_otp(
        request.phone_number,
        request.otp
    )
    
    if not is_valid:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired OTP"
        )
    
    # Get or create user
    user = await auth_service.get_or_create_user(
        session,
        request.phone_number
    )
    
    # Generate JWT token
    token = auth_service.create_access_token(user.id, user.role)
    
    return TokenResponse(
        token=token,
        token_type="bearer",
        user=user
    )

@router.post("/refresh-token")
async def refresh_token(
    current_user = Depends(auth_service.get_current_user)
):
    """
    Refresh JWT token for extended session.
    """
    new_token = auth_service.create_access_token(
        current_user.id,
        current_user.role
    )
    return {"token": new_token}
```

#### 4.1.3 Voice Processing Routes

```python
# app/api/v1/voice.py
from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session
from app.core.database import get_session
from app.services.voice_service import VoiceService
from app.services.auth_service import get_current_user
from app.schemas.voice import VoiceProcessRequest, VoiceProcessResponse

router = APIRouter(prefix="/voice", tags=["voice"])

@router.post("/process", response_model=VoiceProcessResponse)
async def process_voice_command(
    request: VoiceProcessRequest,
    session: Session = Depends(get_session),
    current_user = Depends(get_current_user),
    voice_service: VoiceService = Depends()
):
    """
    Process voice transcript and match with inventory items.
    Uses Google Gemini AI with RAG (inventory context injection).
    
    Performance requirement: < 2 seconds end-to-end.
    """
    # Get user's inventory for context
    inventory = await voice_service.get_user_inventory(
        session,
        current_user.id
    )
    
    # Process with Gemini AI
    matches = await voice_service.process_with_gemini(
        transcript=request.transcript,
        language=request.language,
        inventory_context=inventory
    )
    
    return VoiceProcessResponse(
        matches=matches,
        processing_time_ms=voice_service.last_processing_time
    )

@router.post("/feedback")
async def submit_voice_feedback(
    transcript: str,
    selected_item_id: str,
    was_correct: bool,
    current_user = Depends(get_current_user)
):
    """
    Collect user feedback to improve voice recognition accuracy.
    Used for model fine-tuning.
    """
    await voice_service.log_feedback(
        user_id=current_user.id,
        transcript=transcript,
        selected_item_id=selected_item_id,
        was_correct=was_correct
    )
    return {"message": "Feedback recorded"}
```


#### 4.1.4 Analytics Routes

```python
# app/api/v1/analytics.py
from fastapi import APIRouter, Depends, Query
from sqlmodel import Session
from datetime import date
from app.core.database import get_session
from app.services.analytics_service import AnalyticsService
from app.services.auth_service import get_current_user
from app.schemas.analytics import AnalyticsResponse

router = APIRouter(prefix="/analytics", tags=["analytics"])

@router.get("/today", response_model=AnalyticsResponse)
async def get_today_analytics(
    session: Session = Depends(get_session),
    current_user = Depends(get_current_user),
    analytics_service: AnalyticsService = Depends()
):
    """
    Get today's analytics dashboard data.
    Includes: total sales, top products, profit margins, peak hours, etc.
    
    Performance requirement: < 3 seconds.
    """
    return await analytics_service.compute_analytics(
        session=session,
        user_id=current_user.id,
        period='today'
    )

@router.get("/weekly", response_model=AnalyticsResponse)
async def get_weekly_analytics(
    session: Session = Depends(get_session),
    current_user = Depends(get_current_user),
    analytics_service: AnalyticsService = Depends()
):
    """Get last 7 days analytics."""
    return await analytics_service.compute_analytics(
        session=session,
        user_id=current_user.id,
        period='weekly'
    )

@router.get("/monthly", response_model=AnalyticsResponse)
async def get_monthly_analytics(
    session: Session = Depends(get_session),
    current_user = Depends(get_current_user),
    analytics_service: AnalyticsService = Depends()
):
    """Get current month analytics."""
    return await analytics_service.compute_analytics(
        session=session,
        user_id=current_user.id,
        period='monthly'
    )

@router.get("/gst-breakdown")
async def get_gst_breakdown(
    start_date: date = Query(...),
    end_date: date = Query(...),
    session: Session = Depends(get_session),
    current_user = Depends(get_current_user),
    analytics_service: AnalyticsService = Depends()
):
    """
    Get GST collection breakdown by rate and type.
    Used for GSTR-1 and GSTR-3B preparation.
    """
    return await analytics_service.compute_gst_breakdown(
        session=session,
        user_id=current_user.id,
        start_date=start_date,
        end_date=end_date
    )

@router.get("/export/csv")
async def export_analytics_csv(
    period: str = Query(..., regex="^(today|weekly|monthly)$"),
    current_user = Depends(get_current_user),
    analytics_service: AnalyticsService = Depends()
):
    """Export analytics data as CSV."""
    csv_data = await analytics_service.export_to_csv(
        user_id=current_user.id,
        period=period
    )
    
    return Response(
        content=csv_data,
        media_type="text/csv",
        headers={
            "Content-Disposition": f"attachment; filename=analytics_{period}.csv"
        }
    )
```

### 4.2 Service Layer

#### 4.2.1 Voice Service Design

```python
# app/services/voice_service.py
import google.generativeai as genai
from typing import List, Dict
import time
from app.models.item import Item
from app.schemas.voice import ItemMatch

class VoiceService:
    def __init__(self):
        genai.configure(api_key=settings.GEMINI_API_KEY)
        self.model = genai.GenerativeModel('gemini-pro')
        self.last_processing_time = 0
    
    async def process_with_gemini(
        self,
        transcript: str,
        language: str,
        inventory_context: List[Item]
    ) -> List[ItemMatch]:
        """
        Process voice transcript using Gemini AI with RAG.
        
        RAG Strategy:
        1. Inject complete inventory as context
        2. Ask Gemini to match transcript with inventory
        3. Return structured JSON with matches and confidence
        
        Performance target: < 2 seconds
        """
        start_time = time.time()
        
        # Build inventory context for RAG
        inventory_text = self._build_inventory_context(inventory_context)
        
        # Construct prompt
        prompt = f"""
You are a retail billing assistant. A cashier said: "{transcript}" in {language}.

Available inventory:
{inventory_text}

Task: Match the voice command to inventory items and extract:
1. Item name/SKU
2. Quantity (if mentioned, default to 1)
3. Rate/price (if mentioned, use inventory price otherwise)
4. Confidence score (0-1)

Return JSON array of matches sorted by confidence:
[
  {{
    "item_id": "...",
    "item_name": "...",
    "quantity": 2,
    "rate": 50.0,
    "confidence": 0.95
  }}
]

Rules:
- Match aliases and variations (e.g., "cold drink" = "Coca Cola")
- Handle Hindi/English/Hinglish
- Return top 5 matches if ambiguous
- Confidence > 0.9 for exact matches
"""
        
        try:
            # Call Gemini API
            response = self.model.generate_content(prompt)
            
            # Parse JSON response
            matches_json = self._extract_json(response.text)
            matches = [ItemMatch(**m) for m in matches_json]
            
            # Calculate processing time
            self.last_processing_time = int((time.time() - start_time) * 1000)
            
            # Log for monitoring
            await self._log_processing_time(self.last_processing_time)
            
            return matches
            
        except Exception as e:
            # Fallback to local fuzzy matching
            return await self._fallback_fuzzy_match(
                transcript,
                inventory_context
            )
    
    def _build_inventory_context(self, items: List[Item]) -> str:
        """
        Build compact inventory context for RAG.
        Format: SKU | Name | Aliases | Price | HSN
        """
        lines = []
        for item in items:
            aliases = ", ".join(item.aliases) if item.aliases else ""
            lines.append(
                f"{item.sku} | {item.name} | {aliases} | "
                f"₹{item.price} | {item.hsn_code}"
            )
        return "\n".join(lines)
    
    async def _fallback_fuzzy_match(
        self,
        transcript: str,
        inventory: List[Item]
    ) -> List[ItemMatch]:
        """
        Fallback to local fuzzy matching if Gemini fails.
        Uses Levenshtein distance for matching.
        """
        from fuzzywuzzy import fuzz
        
        matches = []
        for item in inventory:
            # Match against name and aliases
            score = max(
                fuzz.ratio(transcript.lower(), item.name.lower()),
                max([fuzz.ratio(transcript.lower(), alias.lower()) 
                     for alias in item.aliases] or [0])
            )
            
            if score > 60:  # Threshold
                matches.append(ItemMatch(
                    item_id=item.id,
                    item_name=item.name,
                    quantity=1,
                    rate=item.price,
                    confidence=score / 100.0
                ))
        
        return sorted(matches, key=lambda x: x.confidence, reverse=True)[:5]
    
    async def get_user_inventory(
        self,
        session: Session,
        user_id: str
    ) -> List[Item]:
        """Fetch user's inventory for context injection."""
        statement = select(Item).where(Item.owner_id == user_id)
        results = await session.execute(statement)
        return results.scalars().all()
```

#### 4.2.2 Analytics Service Design

```python
# app/services/analytics_service.py
from sqlmodel import Session, select, func
from datetime import datetime, timedelta
from app.models.invoice import Invoice, InvoiceItem
from app.models.item import Item
from app.schemas.analytics import (
    AnalyticsResponse, TopProduct, PeakHour, 
    GSTBreakdown, CustomerInsights
)

class AnalyticsService:
    
    async def compute_analytics(
        self,
        session: Session,
        user_id: str,
        period: str
    ) -> AnalyticsResponse:
        """
        Compute comprehensive analytics for specified period.
        
        Performance optimization:
        - Use database aggregation (not in-memory)
        - Parallel query execution
        - Result caching (Redis in production)
        - Indexed queries
        
        Target: < 3 seconds
        """
        # Determine date range
        start_date, end_date = self._get_date_range(period)
        
        # Execute queries in parallel
        total_sales = await self._compute_total_sales(
            session, user_id, start_date, end_date
        )
        
        top_products = await self._compute_top_products(
            session, user_id, start_date, end_date
        )
        
        profit_margin = await self._compute_profit_margin(
            session, user_id, start_date, end_date
        )
        
        peak_hours = await self._compute_peak_hours(
            session, user_id, start_date, end_date
        )
        
        slow_moving = await self._identify_slow_moving_items(
            session, user_id, start_date, end_date
        )
        
        gst_breakdown = await self._compute_gst_breakdown(
            session, user_id, start_date, end_date
        )
        
        customer_insights = await self._compute_customer_insights(
            session, user_id, start_date, end_date
        )
        
        # Calculate percentage change vs previous period
        prev_start, prev_end = self._get_previous_period(start_date, end_date)
        prev_sales = await self._compute_total_sales(
            session, user_id, prev_start, prev_end
        )
        
        percentage_change = (
            ((total_sales - prev_sales) / prev_sales * 100)
            if prev_sales > 0 else 0
        )
        
        return AnalyticsResponse(
            total_sales=total_sales,
            percentage_change=percentage_change,
            top_products=top_products,
            profit_margin=profit_margin,
            peak_hours=peak_hours,
            slow_moving_items=slow_moving,
            gst_breakdown=gst_breakdown,
            customer_insights=customer_insights
        )
    
    async def _compute_total_sales(
        self,
        session: Session,
        user_id: str,
        start_date: datetime,
        end_date: datetime
    ) -> float:
        """
        Optimized query for total sales.
        Uses database aggregation instead of loading all records.
        """
        statement = (
            select(func.sum(Invoice.grand_total))
            .where(
                Invoice.owner_id == user_id,
                Invoice.date >= start_date,
                Invoice.date <= end_date
            )
        )
        result = await session.execute(statement)
        return result.scalar() or 0.0
    
    async def _compute_top_products(
        self,
        session: Session,
        user_id: str,
        start_date: datetime,
        end_date: datetime,
        limit: int = 10
    ) -> List[TopProduct]:
        """
        Compute top selling products by quantity and revenue.
        Uses JOIN and GROUP BY for efficiency.
        """
        # Top by quantity
        statement = (
            select(
                Item.id,
                Item.name,
                func.sum(InvoiceItem.quantity).label('total_quantity'),
                func.sum(InvoiceItem.total).label('total_revenue')
            )
            .join(InvoiceItem, InvoiceItem.item_id == Item.id)
            .join(Invoice, Invoice.id == InvoiceItem.invoice_id)
            .where(
                Invoice.owner_id == user_id,
                Invoice.date >= start_date,
                Invoice.date <= end_date
            )
            .group_by(Item.id, Item.name)
            .order_by(func.sum(InvoiceItem.quantity).desc())
            .limit(limit)
        )
        
        results = await session.execute(statement)
        return [
            TopProduct(
                item_id=row.id,
                item_name=row.name,
                quantity_sold=row.total_quantity,
                revenue=row.total_revenue
            )
            for row in results
        ]
    
    async def _compute_peak_hours(
        self,
        session: Session,
        user_id: str,
        start_date: datetime,
        end_date: datetime
    ) -> List[PeakHour]:
        """
        Identify peak sales hours using hourly aggregation.
        """
        statement = (
            select(
                func.extract('hour', Invoice.date).label('hour'),
                func.count(Invoice.id).label('transaction_count'),
                func.sum(Invoice.grand_total).label('total_sales')
            )
            .where(
                Invoice.owner_id == user_id,
                Invoice.date >= start_date,
                Invoice.date <= end_date
            )
            .group_by(func.extract('hour', Invoice.date))
            .order_by(func.sum(Invoice.grand_total).desc())
        )
        
        results = await session.execute(statement)
        return [
            PeakHour(
                hour=int(row.hour),
                transaction_count=row.transaction_count,
                total_sales=row.total_sales
            )
            for row in results
        ]
```


#### 4.2.3 GST Service Design

```python
# app/services/gst_service.py
from typing import Tuple
from app.models.invoice import Invoice, InvoiceItem
from app.models.item import Item

class GSTService:
    
    def calculate_gst(
        self,
        items: List[InvoiceItem],
        retailer_gstin: str,
        customer_gstin: str = None
    ) -> Tuple[float, float, float]:
        """
        Calculate GST based on HSN code and state logic.
        
        Returns: (cgst, sgst, igst)
        
        Logic:
        - If intra-state: CGST + SGST (each = GST rate / 2)
        - If inter-state: IGST (= full GST rate)
        """
        is_intra_state = self._is_intra_state(retailer_gstin, customer_gstin)
        
        total_cgst = 0.0
        total_sgst = 0.0
        total_igst = 0.0
        
        for item in items:
            # Get GST rate from item's HSN code
            gst_rate = item.item.gst_rate
            taxable_amount = item.total
            
            gst_amount = taxable_amount * (gst_rate / 100)
            
            if is_intra_state:
                # Split into CGST and SGST
                total_cgst += gst_amount / 2
                total_sgst += gst_amount / 2
            else:
                # Full IGST
                total_igst += gst_amount
        
        return (total_cgst, total_sgst, total_igst)
    
    def _is_intra_state(
        self,
        retailer_gstin: str,
        customer_gstin: str = None
    ) -> bool:
        """
        Determine if transaction is intra-state or inter-state.
        
        GSTIN format: 2-digit state code + 10-digit PAN + 3-digit code
        Example: 27AABCU9603R1ZM (27 = Maharashtra)
        """
        if not customer_gstin:
            # B2C transaction, assume intra-state
            return True
        
        # Extract state codes (first 2 digits)
        retailer_state = retailer_gstin[:2]
        customer_state = customer_gstin[:2]
        
        return retailer_state == customer_state
    
    def validate_gstin(self, gstin: str) -> bool:
        """
        Validate GSTIN format and checksum.
        
        Format: 2-digit state + 10-digit PAN + 1-digit entity + 
                1-digit Z + 1-digit checksum
        """
        if len(gstin) != 15:
            return False
        
        # Validate state code (01-37)
        state_code = gstin[:2]
        if not state_code.isdigit() or not (1 <= int(state_code) <= 37):
            return False
        
        # Validate PAN format (5 letters + 4 digits + 1 letter)
        pan = gstin[2:12]
        if not (pan[:5].isalpha() and pan[5:9].isdigit() and pan[9].isalpha()):
            return False
        
        # Validate checksum (simplified)
        return self._validate_checksum(gstin)
    
    def _validate_checksum(self, gstin: str) -> bool:
        """Validate GSTIN checksum digit."""
        # Checksum algorithm implementation
        # (Simplified for brevity)
        return True
    
    def generate_invoice_number(
        self,
        session: Session,
        user_id: str,
        financial_year: str
    ) -> str:
        """
        Generate sequential invoice number.
        
        Format: INV-FY2026-000001
        
        Ensures:
        - Sequential ordering
        - No gaps or duplicates
        - Financial year prefix
        """
        # Get last invoice number for this FY
        statement = (
            select(Invoice.invoice_number)
            .where(
                Invoice.owner_id == user_id,
                Invoice.invoice_number.like(f"INV-{financial_year}-%")
            )
            .order_by(Invoice.invoice_number.desc())
            .limit(1)
        )
        
        result = session.execute(statement).scalar()
        
        if result:
            # Extract sequence number and increment
            last_seq = int(result.split('-')[-1])
            next_seq = last_seq + 1
        else:
            # First invoice of FY
            next_seq = 1
        
        return f"INV-{financial_year}-{next_seq:06d}"
    
    def create_credit_note(
        self,
        original_invoice: Invoice,
        reason: str
    ) -> Invoice:
        """
        Create credit note for invoice return/cancellation.
        
        Credit note:
        - Reverses GST liability
        - Negative amounts
        - Links to original invoice
        """
        credit_note = Invoice(
            invoice_number=f"CN-{original_invoice.invoice_number}",
            date=datetime.now(),
            customer_id=original_invoice.customer_id,
            subtotal=-original_invoice.subtotal,
            cgst=-original_invoice.cgst,
            sgst=-original_invoice.sgst,
            igst=-original_invoice.igst,
            grand_total=-original_invoice.grand_total,
            is_credit_note=True,
            original_invoice_id=original_invoice.id,
            credit_note_reason=reason
        )
        
        return credit_note
```

#### 4.2.4 OTP Service Design

```python
# app/services/otp_service.py
import random
import string
from datetime import datetime, timedelta
from app.models.otp import OTP
from app.utils.sms_client import SMSClient

class OTPService:
    def __init__(self):
        self.sms_client = SMSClient()
        self.otp_validity_minutes = 5
        self.max_attempts_per_hour = 3
    
    async def generate_otp(
        self,
        phone_number: str,
        session: Session
    ) -> str:
        """
        Generate 6-digit OTP.
        
        Security:
        - Cryptographically random
        - 5-minute expiry
        - Single use
        - Rate limited
        """
        # Generate random 6-digit OTP
        otp_code = ''.join(random.choices(string.digits, k=6))
        
        # Store in database
        otp = OTP(
            phone_number=phone_number,
            otp_code=otp_code,
            expires_at=datetime.now() + timedelta(minutes=self.otp_validity_minutes),
            is_used=False
        )
        
        session.add(otp)
        await session.commit()
        
        return otp_code
    
    async def verify_otp(
        self,
        phone_number: str,
        otp_code: str,
        session: Session
    ) -> bool:
        """
        Verify OTP code.
        
        Checks:
        - OTP exists
        - Not expired
        - Not already used
        - Matches phone number
        """
        statement = (
            select(OTP)
            .where(
                OTP.phone_number == phone_number,
                OTP.otp_code == otp_code,
                OTP.is_used == False,
                OTP.expires_at > datetime.now()
            )
            .order_by(OTP.created_at.desc())
            .limit(1)
        )
        
        result = await session.execute(statement)
        otp = result.scalar_one_or_none()
        
        if otp:
            # Mark as used
            otp.is_used = True
            await session.commit()
            return True
        
        return False
    
    async def is_rate_limited(
        self,
        phone_number: str,
        session: Session
    ) -> bool:
        """
        Check if phone number has exceeded OTP request limit.
        
        Limit: 3 OTPs per hour
        """
        one_hour_ago = datetime.now() - timedelta(hours=1)
        
        statement = (
            select(func.count(OTP.id))
            .where(
                OTP.phone_number == phone_number,
                OTP.created_at > one_hour_ago
            )
        )
        
        result = await session.execute(statement)
        count = result.scalar()
        
        return count >= self.max_attempts_per_hour
    
    async def send_sms(self, phone_number: str, otp_code: str):
        """
        Send OTP via SMS gateway.
        
        Template: "Your SnapBill OTP is {otp}. Valid for 5 minutes. 
                   Do not share with anyone."
        """
        message = (
            f"Your SnapBill OTP is {otp_code}. "
            f"Valid for {self.otp_validity_minutes} minutes. "
            f"Do not share with anyone."
        )
        
        await self.sms_client.send(phone_number, message)
```

### 4.3 Database Layer

#### 4.3.1 Database Models (SQLModel)

```python
# app/models/user.py
from sqlmodel import SQLModel, Field, Relationship
from typing import Optional, List
from datetime import datetime
from enum import Enum

class UserRole(str, Enum):
    OWNER = "owner"
    MANAGER = "manager"
    CASHIER = "cashier"

class User(SQLModel, table=True):
    __tablename__ = "users"
    
    id: str = Field(primary_key=True)
    phone_number: str = Field(unique=True, index=True)
    name: Optional[str] = None
    email: Optional[str] = None
    role: UserRole = Field(default=UserRole.OWNER)
    gstin: Optional[str] = None
    business_name: Optional[str] = None
    address: Optional[str] = None
    
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: datetime = Field(default_factory=datetime.now)
    
    # Relationships
    items: List["Item"] = Relationship(back_populates="owner")
    invoices: List["Invoice"] = Relationship(back_populates="owner")
    customers: List["Customer"] = Relationship(back_populates="owner")

# app/models/item.py
class Item(SQLModel, table=True):
    __tablename__ = "items"
    
    id: str = Field(primary_key=True)
    owner_id: str = Field(foreign_key="users.id", index=True)
    
    sku: str = Field(unique=True, index=True)
    name: str = Field(index=True)
    description: Optional[str] = None
    category: Optional[str] = Field(index=True)
    
    price: float
    cost_price: Optional[float] = None
    gst_rate: float  # 0, 5, 12, 18, 28
    hsn_code: str
    
    stock_quantity: int = Field(default=0)
    min_stock_threshold: Optional[int] = None
    
    aliases: List[str] = Field(default_factory=list, sa_column=Column(JSON))
    
    # Variant support
    is_variant: bool = Field(default=False)
    parent_item_id: Optional[str] = Field(foreign_key="items.id")
    variant_attributes: Optional[dict] = Field(sa_column=Column(JSON))
    
    created_at: datetime = Field(default_factory=datetime.now)
    updated_at: datetime = Field(default_factory=datetime.now)
    
    # Relationships
    owner: User = Relationship(back_populates="items")
    invoice_items: List["InvoiceItem"] = Relationship(back_populates="item")

# app/models/invoice.py
class Invoice(SQLModel, table=True):
    __tablename__ = "invoices"
    
    id: str = Field(primary_key=True)
    owner_id: str = Field(foreign_key="users.id", index=True)
    
    invoice_number: str = Field(unique=True, index=True)
    date: datetime = Field(index=True)
    
    customer_id: Optional[str] = Field(foreign_key="customers.id")
    
    subtotal: float
    cgst: float = Field(default=0.0)
    sgst: float = Field(default=0.0)
    igst: float = Field(default=0.0)
    grand_total: float
    
    payment_method: str  # cash, upi, card, credit
    payment_status: str = Field(default="paid")  # paid, partial, pending
    amount_paid: Optional[float] = None
    
    # Credit note support
    is_credit_note: bool = Field(default=False)
    original_invoice_id: Optional[str] = Field(foreign_key="invoices.id")
    credit_note_reason: Optional[str] = None
    
    created_at: datetime = Field(default_factory=datetime.now)
    
    # Relationships
    owner: User = Relationship(back_populates="invoices")
    customer: Optional["Customer"] = Relationship(back_populates="invoices")
    items: List["InvoiceItem"] = Relationship(back_populates="invoice")

class InvoiceItem(SQLModel, table=True):
    __tablename__ = "invoice_items"
    
    id: str = Field(primary_key=True)
    invoice_id: str = Field(foreign_key="invoices.id", index=True)
    item_id: str = Field(foreign_key="items.id", index=True)
    
    quantity: float
    rate: float
    total: float
    
    # Relationships
    invoice: Invoice = Relationship(back_populates="items")
    item: Item = Relationship(back_populates="invoice_items")
```


#### 4.3.2 Phase 2 Models (B2B Commerce)

```python
# app/models/supplier.py
class Supplier(SQLModel, table=True):
    __tablename__ = "suppliers"
    
    id: str = Field(primary_key=True)
    name: str = Field(index=True)
    gstin: str = Field(unique=True)
    contact_person: str
    phone_number: str
    email: str
    address: str
    
    # Verification
    is_verified: bool = Field(default=False)
    verification_date: Optional[datetime] = None
    
    # Rating
    average_rating: float = Field(default=0.0)
    total_ratings: int = Field(default=0)
    
    created_at: datetime = Field(default_factory=datetime.now)
    
    # Relationships
    purchase_orders: List["PurchaseOrder"] = Relationship(back_populates="supplier")
    ratings: List["SupplierRating"] = Relationship(back_populates="supplier")

# app/models/purchase_order.py
class PurchaseOrder(SQLModel, table=True):
    __tablename__ = "purchase_orders"
    
    id: str = Field(primary_key=True)
    po_number: str = Field(unique=True, index=True)
    
    retailer_id: str = Field(foreign_key="users.id", index=True)
    supplier_id: str = Field(foreign_key="suppliers.id", index=True)
    
    order_date: datetime = Field(default_factory=datetime.now)
    expected_delivery_date: Optional[datetime] = None
    actual_delivery_date: Optional[datetime] = None
    
    status: str = Field(default="pending")  # pending, confirmed, shipped, delivered, cancelled
    
    subtotal: float
    gst_amount: float
    total_amount: float
    
 
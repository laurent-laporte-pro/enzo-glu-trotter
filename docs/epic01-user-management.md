# EPIC 1 : Gestion des utilisateurs

Je vais vous aider à créer une application Flutter simple pour gérer les utilisateurs en local avec SQLite.
Je vais décomposer cela en plusieurs parties.


```dart
// lib/models/enums.dart
enum ScreenTheme { LIGHT, DARK, SYSTEM }

enum UserLanguage { EN, FR }

// lib/models/geolocation.dart
class Geolocation {
  final double latitude;
  final double longitude;
  final double? altitude;
  final String? address;
  final DateTime timestamp;

  Geolocation({
    required this.latitude,
    required this.longitude,
    this.altitude,
    this.address,
    required this.timestamp,
  });

  Map<String, dynamic> toMap() {
    return {
      'latitude': latitude,
      'longitude': longitude,
      'altitude': altitude,
      'address': address,
      'timestamp': timestamp.toIso8601String(),
    };
  }

  factory Geolocation.fromMap(Map<String, dynamic> map) {
    return Geolocation(
      latitude: map['latitude'],
      longitude: map['longitude'],
      altitude: map['altitude'],
      address: map['address'],
      timestamp: DateTime.parse(map['timestamp']),
    );
  }
}

// lib/models/user.dart
import 'package:uuid/uuid.dart';
import 'package:crypto/crypto.dart';
import 'dart:convert';

class User {
  final String id;
  String username;
  String email;
  String hashedPassword;
  String firstName;
  String lastName;
  DateTime dateJoined;
  DateTime lastLogin;
  ScreenTheme screenTheme;
  UserLanguage language;
  Geolocation? geolocation;

  User({
    String? id,
    required this.username,
    required this.email,
    required this.hashedPassword,
    required this.firstName,
    required this.lastName,
    DateTime? dateJoined,
    DateTime? lastLogin,
    this.screenTheme = ScreenTheme.SYSTEM,
    this.language = UserLanguage.FR,
    this.geolocation,
  }) : 
    this.id = id ?? Uuid().v4(),
    this.dateJoined = dateJoined ?? DateTime.now(),
    this.lastLogin = lastLogin ?? DateTime.now();

  static String hashPassword(String password) {
    var bytes = utf8.encode(password);
    return sha256.convert(bytes).toString();
  }

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'username': username,
      'email': email,
      'hashed_password': hashedPassword,
      'first_name': firstName,
      'last_name': lastName,
      'date_joined': dateJoined.toIso8601String(),
      'last_login': lastLogin.toIso8601String(),
      'screen_theme': screenTheme.index,
      'language': language.index,
      'geolocation': geolocation?.toMap(),
    };
  }

  factory User.fromMap(Map<String, dynamic> map) {
    return User(
      id: map['id'],
      username: map['username'],
      email: map['email'],
      hashedPassword: map['hashed_password'],
      firstName: map['first_name'],
      lastName: map['last_name'],
      dateJoined: DateTime.parse(map['date_joined']),
      lastLogin: DateTime.parse(map['last_login']),
      screenTheme: ScreenTheme.values[map['screen_theme']],
      language: UserLanguage.values[map['language']],
      geolocation: map['geolocation'] != null 
        ? Geolocation.fromMap(map['geolocation'])
        : null,
    );
  }
}

// lib/database/database_helper.dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class DatabaseHelper {
  static final DatabaseHelper _instance = DatabaseHelper._internal();
  static Database? _database;

  factory DatabaseHelper() => _instance;

  DatabaseHelper._internal();

  Future<Database> get database async {
    if (_database != null) return _database!;
    _database = await _initDatabase();
    return _database!;
  }

  Future<Database> _initDatabase() async {
    String path = join(await getDatabasesPath(), 'user_database.db');
    return await openDatabase(
      path,
      version: 1,
      onCreate: _onCreate,
    );
  }

  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE users(
        id TEXT PRIMARY KEY,
        username TEXT UNIQUE,
        email TEXT UNIQUE,
        hashed_password TEXT,
        first_name TEXT,
        last_name TEXT,
        date_joined TEXT,
        last_login TEXT,
        screen_theme INTEGER,
        language INTEGER,
        geolocation TEXT
      )
    ''');
  }

  Future<void> insertUser(User user) async {
    final Database db = await database;
    await db.insert(
      'users',
      user.toMap(),
      conflictAlgorithm: ConflictAlgorithm.replace,
    );
  }

  Future<User?> getUser(String id) async {
    final Database db = await database;
    final List<Map<String, dynamic>> maps = await db.query(
      'users',
      where: 'id = ?',
      whereArgs: [id],
    );

    if (maps.isEmpty) return null;
    return User.fromMap(maps.first);
  }

  Future<void> updateUser(User user) async {
    final db = await database;
    await db.update(
      'users',
      user.toMap(),
      where: 'id = ?',
      whereArgs: [user.id],
    );
  }

  Future<void> deleteUser(String id) async {
    final db = await database;
    await db.delete(
      'users',
      where: 'id = ?',
      whereArgs: [id],
    );
  }
}

```

Pour utiliser ce code, vous devrez ajouter les dépendances suivantes dans votre `pubspec.yaml` :

```yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite: ^2.3.0
  path: ^1.8.3
  uuid: ^4.0.0
  crypto: ^3.0.3
```

Ce code fournit :
1. Les modèles de données (enums, Geolocation, User)
2. La gestion de la base de données SQLite avec les opérations CRUD
3. Le hachage sécurisé des mots de passe
4. La conversion des objets en format Map pour le stockage

Voulez-vous que je crée maintenant les interfaces utilisateur pour :
- L'inscription (sign up)
- L'affichage du profil
- La modification du profil
- La suppression du compte

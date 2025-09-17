# Sequelize
A comprehensive guide/cheatsheet for learning sequelize
# Sequelize Learning Guide

A comprehensive guide to learning Sequelize, the popular Object-Relational Mapping (ORM) library for Node.js.

## Table of Contents

1. [Introduction](#introduction)
2. [Installation and Setup](#installation-and-setup)
3. [Database Connection](#database-connection)
4. [Models](#models)
5. [Data Types](#data-types)
6. [Model Associations](#model-associations)
7. [Migrations](#migrations)
8. [Seeders](#seeders)
9. [Querying Data](#querying-data)
10. [Advanced Queries](#advanced-queries)
11. [Transactions](#transactions)
12. [Hooks and Validations](#hooks-and-validations)
13. [CLI Usage](#cli-usage)
14. [Best Practices](#best-practices)
15. [Performance Optimization](#performance-optimization)
16. [Testing](#testing)
17. [Common Patterns](#common-patterns)
18. [Troubleshooting](#troubleshooting)

## Introduction

Sequelize is a promise-based Node.js ORM for Postgres, MySQL, MariaDB, SQLite, and Microsoft SQL Server. It features solid transaction support, relations, eager and lazy loading, read replication, and more.

### Why Use Sequelize?

- **Database Abstraction**: Write database-agnostic code
- **Model Definitions**: Define your data structure in JavaScript
- **Migrations**: Version control for your database schema
- **Associations**: Easy relationship management
- **Validation**: Built-in data validation
- **Hooks**: Lifecycle events for models
- **Transactions**: ACID transaction support

## Installation and Setup

### Basic Installation

```bash
# Install Sequelize
npm install sequelize

# Install database driver (choose one)
npm install pg pg-hstore      # PostgreSQL
npm install mysql2            # MySQL
npm install mariadb          # MariaDB
npm install sqlite3          # SQLite
npm install mssql            # Microsoft SQL Server
```

### CLI Installation

```bash
# Install Sequelize CLI globally
npm install -g sequelize-cli

# Or locally
npm install --save-dev sequelize-cli
```

### Project Initialization

```bash
# Initialize Sequelize project structure
npx sequelize-cli init
```

This creates:
- `config/config.json` - Database configuration
- `models/` - Model definitions
- `migrations/` - Database migrations
- `seeders/` - Database seeders

## Database Connection

### Basic Connection

```javascript
const { Sequelize } = require('sequelize');

// Option 1: Database URL
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');

// Option 2: Separate parameters
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres', // 'mysql' | 'mariadb' | 'postgres' | 'mssql' | 'sqlite'
  port: 5432,
  logging: console.log, // Set to false to disable logging
});

// Test connection
async function testConnection() {
  try {
    await sequelize.authenticate();
    console.log('Connection established successfully.');
  } catch (error) {
    console.error('Unable to connect to database:', error);
  }
}
```

### Connection Pooling

```javascript
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres',
  pool: {
    max: 20,      // Maximum connections in pool
    min: 0,       // Minimum connections in pool
    acquire: 30000, // Maximum time to get connection
    idle: 10000   // Maximum time connection can be idle
  }
});
```

### Environment Configuration

```javascript
// config/database.js
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  process.env.DB_NAME || 'myapp_development',
  process.env.DB_USER || 'root',
  process.env.DB_PASS || '',
  {
    host: process.env.DB_HOST || 'localhost',
    dialect: process.env.DB_DIALECT || 'postgres',
    port: process.env.DB_PORT || 5432,
    logging: process.env.NODE_ENV === 'development' ? console.log : false,
  }
);

module.exports = sequelize;
```

## Models

### Defining Models

#### Method 1: Using `sequelize.define()`

```javascript
const { DataTypes } = require('sequelize');

const User = sequelize.define('User', {
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  firstName: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notEmpty: true,
      len: [2, 50]
    }
  },
  lastName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true
    }
  },
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: 0,
      max: 120
    }
  },
  isActive: {
    type: DataTypes.BOOLEAN,
    defaultValue: true
  }
}, {
  // Model options
  tableName: 'users',
  timestamps: true, // adds createdAt and updatedAt
  paranoid: true,   // adds deletedAt (soft delete)
  underscored: true // use snake_case for columns
});
```

#### Method 2: Extending Model Class

```javascript
const { Model, DataTypes } = require('sequelize');

class User extends Model {
  // Instance methods
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
  
  // Class methods
  static findByEmail(email) {
    return this.findOne({ where: { email } });
  }
}

User.init({
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  firstName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING,
    allowNull: false
  },
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true
  }
}, {
  sequelize,
  modelName: 'User',
  tableName: 'users'
});
```

### Model Options

```javascript
const User = sequelize.define('User', {
  // attributes
}, {
  // Options
  tableName: 'custom_users',     // Custom table name
  timestamps: true,              // Add createdAt/updatedAt
  paranoid: true,                // Soft delete with deletedAt
  underscored: true,             // Use snake_case
  freezeTableName: true,         // Prevent pluralization
  version: true,                 // Add version column
  indexes: [                     // Define indexes
    {
      fields: ['email']
    },
    {
      fields: ['firstName', 'lastName'],
      name: 'user_name_index'
    }
  ],
  hooks: {                       // Lifecycle hooks
    beforeCreate: (user) => {
      user.email = user.email.toLowerCase();
    }
  }
});
```

## Data Types

### Common Data Types

```javascript
const { DataTypes } = require('sequelize');

const Example = sequelize.define('Example', {
  // String types
  shortString: DataTypes.STRING,           // VARCHAR(255)
  longString: DataTypes.STRING(1000),     // VARCHAR(1000)
  text: DataTypes.TEXT,                   // TEXT
  mediumText: DataTypes.TEXT('medium'),   // MEDIUMTEXT
  longText: DataTypes.TEXT('long'),       // LONGTEXT
  
  // Number types
  integer: DataTypes.INTEGER,             // INTEGER
  bigInteger: DataTypes.BIGINT,           // BIGINT
  float: DataTypes.FLOAT,                 // FLOAT
  real: DataTypes.REAL,                   // REAL
  double: DataTypes.DOUBLE,               // DOUBLE
  decimal: DataTypes.DECIMAL(10, 2),      // DECIMAL(10,2)
  
  // Boolean
  isActive: DataTypes.BOOLEAN,            // BOOLEAN
  
  // Date types
  birthday: DataTypes.DATEONLY,           // DATE
  appointmentTime: DataTypes.DATE,        // DATETIME
  createdTime: DataTypes.TIME,            // TIME
  
  // JSON (PostgreSQL, MySQL, MariaDB, SQLite)
  metadata: DataTypes.JSON,
  settings: DataTypes.JSONB,              // PostgreSQL only
  
  // Arrays (PostgreSQL only)
  tags: DataTypes.ARRAY(DataTypes.STRING),
  
  // UUID
  uuid: {
    type: DataTypes.UUID,
    defaultValue: DataTypes.UUIDV4
  },
  
  // ENUM
  status: {
    type: DataTypes.ENUM('active', 'inactive', 'pending'),
    defaultValue: 'pending'
  },
  
  // Virtual fields (not stored in DB)
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      const parts = value.split(' ');
      this.setDataValue('firstName', parts[0]);
      this.setDataValue('lastName', parts[1]);
    }
  }
});
```

### Custom Getters and Setters

```javascript
const User = sequelize.define('User', {
  email: {
    type: DataTypes.STRING,
    set(value) {
      // Always store email in lowercase
      this.setDataValue('email', value.toLowerCase());
    },
    get() {
      // Always return email in lowercase
      return this.getDataValue('email')?.toLowerCase();
    }
  },
  password: {
    type: DataTypes.STRING,
    set(value) {
      // Hash password before storing
      const bcrypt = require('bcrypt');
      const hashedPassword = bcrypt.hashSync(value, 10);
      this.setDataValue('password', hashedPassword);
    }
  }
});
```

## Model Associations

### One-to-One (hasOne / belongsTo)

```javascript
// User has one Profile
User.hasOne(Profile, {
  foreignKey: 'userId',
  as: 'profile'
});

Profile.belongsTo(User, {
  foreignKey: 'userId',
  as: 'user'
});
```

### One-to-Many (hasMany / belongsTo)

```javascript
// User has many Posts
User.hasMany(Post, {
  foreignKey: 'authorId',
  as: 'posts'
});

Post.belongsTo(User, {
  foreignKey: 'authorId',
  as: 'author'
});
```

### Many-to-Many (belongsToMany)

```javascript
// User belongs to many Roles through UserRoles
User.belongsToMany(Role, {
  through: 'UserRoles',
  foreignKey: 'userId',
  otherKey: 'roleId',
  as: 'roles'
});

Role.belongsToMany(User, {
  through: 'UserRoles',
  foreignKey: 'roleId',
  otherKey: 'userId',
  as: 'users'
});

// With custom through model
const UserRole = sequelize.define('UserRole', {
  userId: DataTypes.INTEGER,
  roleId: DataTypes.INTEGER,
  assignedAt: DataTypes.DATE,
  assignedBy: DataTypes.INTEGER
});

User.belongsToMany(Role, {
  through: UserRole,
  foreignKey: 'userId',
  as: 'roles'
});
```

### Self-Associations

```javascript
// User can have manager (self-referencing)
User.belongsTo(User, {
  foreignKey: 'managerId',
  as: 'manager'
});

User.hasMany(User, {
  foreignKey: 'managerId',
  as: 'subordinates'
});

// Category tree structure
Category.belongsTo(Category, {
  foreignKey: 'parentId',
  as: 'parent'
});

Category.hasMany(Category, {
  foreignKey: 'parentId',
  as: 'children'
});
```

## Migrations

### Creating Migrations

```bash
# Create a new migration
npx sequelize-cli migration:generate --name create-users-table

# Create model with migration
npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string
```

### Migration Structure

```javascript
// migrations/20231201000000-create-users.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      firstName: {
        type: Sequelize.STRING,
        allowNull: false
      },
      lastName: {
        type: Sequelize.STRING,
        allowNull: false
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true
      },
      isActive: {
        type: Sequelize.BOOLEAN,
        defaultValue: true
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
    
    // Add indexes
    await queryInterface.addIndex('Users', ['email']);
    await queryInterface.addIndex('Users', ['firstName', 'lastName']);
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Users');
  }
};
```

### Advanced Migration Operations

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Add column
    await queryInterface.addColumn('Users', 'phoneNumber', {
      type: Sequelize.STRING,
      allowNull: true
    });
    
    // Change column
    await queryInterface.changeColumn('Users', 'email', {
      type: Sequelize.STRING(100),
      allowNull: false,
      unique: true
    });
    
    // Rename column
    await queryInterface.renameColumn('Users', 'firstName', 'first_name');
    
    // Add foreign key
    await queryInterface.addConstraint('Posts', {
      fields: ['authorId'],
      type: 'foreign key',
      name: 'posts_author_fkey',
      references: {
        table: 'Users',
        field: 'id'
      },
      onDelete: 'CASCADE',
      onUpdate: 'CASCADE'
    });
    
    // Add index
    await queryInterface.addIndex('Users', {
      fields: ['email'],
      unique: true,
      name: 'users_email_unique_idx'
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.removeColumn('Users', 'phoneNumber');
    await queryInterface.renameColumn('Users', 'first_name', 'firstName');
    await queryInterface.removeConstraint('Posts', 'posts_author_fkey');
    await queryInterface.removeIndex('Users', 'users_email_unique_idx');
  }
};
```

### Running Migrations

```bash
# Run all pending migrations
npx sequelize-cli db:migrate

# Undo last migration
npx sequelize-cli db:migrate:undo

# Undo all migrations
npx sequelize-cli db:migrate:undo:all

# Undo to specific migration
npx sequelize-cli db:migrate:undo:all --to 20231201000000-create-users.js
```

## Seeders

### Creating Seeders

```bash
# Generate seeder file
npx sequelize-cli seed:generate --name demo-users
```

### Seeder Structure

```javascript
// seeders/20231201000000-demo-users.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert('Users', [
      {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        isActive: true,
        createdAt: new Date(),
        updatedAt: new Date()
      },
      {
        firstName: 'Jane',
        lastName: 'Smith',
        email: 'jane@example.com',
        isActive: true,
        createdAt: new Date(),
        updatedAt: new Date()
      }
    ]);
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete('Users', null, {});
  }
};
```

### Running Seeders

```bash
# Run all seeders
npx sequelize-cli db:seed:all

# Run specific seeder
npx sequelize-cli db:seed --seed 20231201000000-demo-users.js

# Undo seeders
npx sequelize-cli db:seed:undo:all
```

## Querying Data

### Basic CRUD Operations

#### Create

```javascript
// Create single record
const user = await User.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com'
});

// Create multiple records
const users = await User.bulkCreate([
  { firstName: 'John', lastName: 'Doe', email: 'john@example.com' },
  { firstName: 'Jane', lastName: 'Smith', email: 'jane@example.com' }
]);

// Create with associations
const userWithProfile = await User.create({
  firstName: 'John',
  lastName: 'Doe',
  email: 'john@example.com',
  Profile: {
    bio: 'Software Developer',
    avatar: 'avatar.jpg'
  }
}, {
  include: [{ model: Profile, as: 'profile' }]
});
```

#### Read

```javascript
// Find by primary key
const user = await User.findByPk(1);

// Find one record
const user = await User.findOne({
  where: { email: 'john@example.com' }
});

// Find all records
const users = await User.findAll();

// Find with conditions
const activeUsers = await User.findAll({
  where: {
    isActive: true,
    age: {
      [Op.gte]: 18
    }
  }
});

// Find and count
const { count, rows } = await User.findAndCountAll({
  where: { isActive: true },
  limit: 10,
  offset: 20
});
```

#### Update

```javascript
// Update single record
const user = await User.findByPk(1);
await user.update({
  firstName: 'Jane',
  email: 'jane@example.com'
});

// Update multiple records
await User.update(
  { isActive: false },
  {
    where: {
      lastLogin: {
        [Op.lt]: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) // 30 days ago
      }
    }
  }
);

// Update with increment
await User.increment('loginCount', {
  where: { id: 1 }
});
```

#### Delete

```javascript
// Delete single record
const user = await User.findByPk(1);
await user.destroy();

// Delete multiple records
await User.destroy({
  where: {
    isActive: false
  }
});

// Force delete (ignore paranoid)
await user.destroy({ force: true });
```

### Query Operators

```javascript
const { Op } = require('sequelize');

const users = await User.findAll({
  where: {
    // Basic operators
    id: 1,                          // id = 1
    firstName: 'John',              // firstName = 'John'
    
    // Comparison operators
    age: {
      [Op.gte]: 18,                 // age >= 18
      [Op.lt]: 65                   // age < 65
    },
    
    // String operators
    firstName: {
      [Op.like]: 'J%',              // firstName LIKE 'J%'
      [Op.iLike]: '%john%',         // firstName ILIKE '%john%' (case insensitive)
      [Op.startsWith]: 'Jo',        // firstName LIKE 'Jo%'
      [Op.endsWith]: 'hn',          // firstName LIKE '%hn'
      [Op.substring]: 'oh'          // firstName LIKE '%oh%'
    },
    
    // Array operators
    id: {
      [Op.in]: [1, 2, 3],           // id IN (1, 2, 3)
      [Op.notIn]: [4, 5, 6]         // id NOT IN (4, 5, 6)
    },
    
    // Null checks
    middleName: {
      [Op.is]: null,                // middleName IS NULL
      [Op.not]: null                // middleName IS NOT NULL
    },
    
    // Logical operators
    [Op.and]: [
      { isActive: true },
      { age: { [Op.gte]: 18 } }
    ],
    [Op.or]: [
      { firstName: 'John' },
      { lastName: 'Doe' }
    ]
  }
});
```

### Query Options

```javascript
const users = await User.findAll({
  // Attributes (columns to select)
  attributes: ['id', 'firstName', 'lastName'],
  
  // Exclude attributes
  attributes: {
    exclude: ['password', 'createdAt', 'updatedAt']
  },
  
  // Include calculated fields
  attributes: {
    include: [
      [Sequelize.fn('UPPER', Sequelize.col('firstName')), 'upperFirstName'],
      [Sequelize.literal('CONCAT(firstName, " ", lastName)'), 'fullName']
    ]
  },
  
  // Ordering
  order: [
    ['firstName', 'ASC'],
    ['createdAt', 'DESC'],
    [Sequelize.fn('LENGTH', Sequelize.col('lastName')), 'DESC']
  ],
  
  // Pagination
  limit: 10,
  offset: 20,
  
  // Grouping
  group: ['departmentId'],
  
  // Having (use with group)
  having: Sequelize.where(
    Sequelize.fn('COUNT', Sequelize.col('id')), 
    Op.gt, 
    1
  ),
  
  // Distinct
  distinct: true,
  
  // Subquery
  subQuery: false,
  
  // Raw query
  raw: true,
  
  // Logging
  logging: console.log
});
```

## Advanced Queries

### Eager Loading (Includes)

```javascript
// Basic include
const users = await User.findAll({
  include: [Profile]
});

// Include with alias
const users = await User.findAll({
  include: [{
    model: Profile,
    as: 'profile'
  }]
});

// Include with conditions
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    where: { published: true }
  }]
});

// Nested includes
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    include: [{
      model: Comment,
      as: 'comments',
      include: [{
        model: User,
        as: 'author'
      }]
    }]
  }]
});

// Include all associations
const users = await User.findAll({
  include: { all: true }
});

// Include with separate queries (avoid N+1)
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    separate: true
  }]
});
```

### Aggregation

```javascript
// Count
const userCount = await User.count({
  where: { isActive: true }
});

// Sum
const totalAge = await User.sum('age', {
  where: { isActive: true }
});

// Average
const averageAge = await User.aggregate('age', 'avg', {
  where: { isActive: true }
});

// Min/Max
const minAge = await User.min('age');
const maxAge = await User.max('age');

// Group by with aggregation
const usersByDepartment = await User.findAll({
  attributes: [
    'departmentId',
    [Sequelize.fn('COUNT', Sequelize.col('id')), 'userCount'],
    [Sequelize.fn('AVG', Sequelize.col('age')), 'averageAge']
  ],
  group: ['departmentId'],
  having: Sequelize.where(
    Sequelize.fn('COUNT', Sequelize.col('id')), 
    Op.gt, 
    5
  )
});
```

### Raw Queries

```javascript
// Simple raw query
const users = await sequelize.query(
  "SELECT * FROM users WHERE age > :minAge",
  {
    replacements: { minAge: 18 },
    type: QueryTypes.SELECT
  }
);

// Raw query with model mapping
const users = await sequelize.query(
  "SELECT * FROM users WHERE age > ?",
  {
    replacements: [18],
    model: User,
    mapToModel: true
  }
);

// Raw query for complex operations
const result = await sequelize.query(`
  SELECT 
    u.id,
    u.firstName,
    u.lastName,
    COUNT(p.id) as postCount,
    AVG(p.views) as averageViews
  FROM users u
  LEFT JOIN posts p ON u.id = p.authorId
  WHERE u.isActive = true
  GROUP BY u.id, u.firstName, u.lastName
  HAVING COUNT(p.id) > 0
  ORDER BY postCount DESC
  LIMIT 10
`, {
  type: QueryTypes.SELECT
});
```

### Scopes

```javascript
// Define scopes in model
const User = sequelize.define('User', {
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,
  age: DataTypes.INTEGER,
  isActive: DataTypes.BOOLEAN
}, {
  scopes: {
    // Default scope
    defaultScope: {
      where: { isActive: true }
    },
    
    // Named scopes
    adults: {
      where: { age: { [Op.gte]: 18 } }
    },
    
    withPosts: {
      include: [{
        model: Post,
        as: 'posts'
      }]
    },
    
    // Parameterized scopes
    olderThan: (age) => ({
      where: {
        age: { [Op.gt]: age }
      }
    }),
    
    // Function scopes
    recentlyActive: function(days) {
      return {
        where: {
          lastLoginAt: {
            [Op.gte]: new Date(Date.now() - days * 24 * 60 * 60 * 1000)
          }
        }
      }
    }
  }
});

// Using scopes
const adults = await User.scope('adults').findAll();
const adultsWithPosts = await User.scope(['adults', 'withPosts']).findAll();
const seniors = await User.scope({ method: ['olderThan', 65] }).findAll();
const recentUsers = await User.scope({ method: ['recentlyActive', 7] }).findAll();

// Remove default scope
const allUsers = await User.unscoped().findAll();
```

## Transactions

### Auto-Managed Transactions

```javascript
// Sequelize will automatically commit or rollback
const result = await sequelize.transaction(async (t) => {
  const user = await User.create({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  }, { transaction: t });

  const profile = await Profile.create({
    userId: user.id,
    bio: 'Software Developer'
  }, { transaction: t });

  return { user, profile };
});
```

### Manually Managed Transactions

```javascript
const t = await sequelize.transaction();

try {
  const user = await User.create({
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com'
  }, { transaction: t });

  const profile = await Profile.create({
    userId: user.id,
    bio: 'Software Developer'
  }, { transaction: t });

  await t.commit();
  return { user, profile };
} catch (error) {
  await t.rollback();
  throw error;
}
```

### Transaction Isolation Levels

```javascript
const { Transaction } = require('sequelize');

// Set isolation level
await sequelize.transaction({
  isolationLevel: Transaction.ISOLATION_LEVELS.SERIALIZABLE
}, async (t) => {
  // Transaction operations
});

// Available isolation levels:
// - READ_UNCOMMITTED
// - READ_COMMITTED
// - REPEATABLE_READ
// - SERIALIZABLE
```

### Concurrent Transactions

```javascript
// Using locks
await sequelize.transaction(async (t) => {
  const user = await User.findByPk(1, {
    lock: t.LOCK.UPDATE,
    transaction: t
  });
  
  user.balance += 100;
  await user.save({ transaction: t });
});

// Pessimistic locking
const user = await User.findByPk(1, {
  lock: true,
  transaction: t
});

// Optimistic locking (using version field)
const User = sequelize.define('User', {
  name: DataTypes.STRING,
  version: DataTypes.INTEGER
}, {
  version: true
});
```

## Hooks and Validations

### Model Validations

```javascript
const User = sequelize.define('User', {
  firstName: {
    type: DataTypes.STRING,
    allowNull: false,
    validate: {
      notEmpty: {
        msg: 'First name cannot be empty'
      },
      len: {
        args: [2, 50],
        msg: 'First name must be between 2 and 50 characters'
      }
    }
  },
  
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: {
        msg: 'Must be a valid email address'
      },
      notNull: {
        msg: 'Email is required'
      }
    }
  },
  
  age: {
    type: DataTypes.INTEGER,
    validate: {
      min: {
        args: [0],
        msg: 'Age cannot be negative'
      },
      max: {
        args: [120],
        msg: 'Age cannot exceed 120'
      },
      isInt: {
        msg: 'Age must be an integer'
      }
    }
  },
  
        password: {
    type: DataTypes.STRING,
    validate: {
      len: {
        args: [8, 100],
        msg: 'Password must be between 8 and 100 characters'
      },
      isStrongPassword(value) {
        if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          throw new Error('Password must contain at least one lowercase letter, one uppercase letter, and one digit');
        }
      }
    }
  },
  
  website: {
    type: DataTypes.STRING,
    validate: {
      isUrl: {
        msg: 'Must be a valid URL'
      }
    }
  },
  
  phoneNumber: {
    type: DataTypes.STRING,
    validate: {
      is: {
        args: /^\+?[\d\s\-\(\)]+$/,
        msg: 'Invalid phone number format'
      }
    }
  }
}, {
  // Model-level validations
  validate: {
    // Custom validation function
    bothCoordsOrNone() {
      if ((this.latitude === null) !== (this.longitude === null)) {
        throw new Error('Either both latitude and longitude, or neither!');
      }
    },
    
    // Async validation
    async emailNotExists() {
      const existingUser = await User.findOne({ 
        where: { 
          email: this.email,
          id: { [Op.ne]: this.id || 0 }
        }
      });
      if (existingUser) {
        throw new Error('Email already exists');
      }
    }
  }
});

// Custom validator functions
const isValidUsername = (value) => {
  if (!/^[a-zA-Z0-9_]+$/.test(value)) {
    throw new Error('Username can only contain letters, numbers, and underscores');
  }
};

// Built-in validators:
// - is: pattern matching
// - not: inverse pattern matching
// - isEmail, isUrl, isIP, isAlpha, isAlphanumeric
// - len: string/array length
// - min, max: number range
// - notNull, notEmpty
// - equals: exact match
// - contains, notContains
// - isIn, notIn: value in/not in array
// - isDate, isAfter, isBefore
// - isCreditCard, isUUID
```

### Hooks (Lifecycle Events)

```javascript
const User = sequelize.define('User', {
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,
  email: DataTypes.STRING,
  password: DataTypes.STRING
}, {
  hooks: {
    // Before validation
    beforeValidate: (user, options) => {
      if (user.email) {
        user.email = user.email.toLowerCase().trim();
      }
    },
    
    // After validation
    afterValidate: (user, options) => {
      console.log('User validated successfully');
    },
    
    // Before creation
    beforeCreate: async (user, options) => {
      // Hash password
      if (user.password) {
        const bcrypt = require('bcrypt');
        user.password = await bcrypt.hash(user.password, 10);
      }
      
      // Set default values
      user.isActive = true;
      user.lastLoginAt = new Date();
    },
    
    // After creation
    afterCreate: async (user, options) => {
      // Send welcome email
      await sendWelcomeEmail(user.email);
      
      // Create user profile
      await Profile.create({
        userId: user.id,
        bio: 'New user'
      }, { transaction: options.transaction });
    },
    
    // Before update
    beforeUpdate: (user, options) => {
      if (user.changed('email')) {
        user.emailVerified = false;
      }
    },
    
    // After update
    afterUpdate: async (user, options) => {
      if (user.changed('email')) {
        await sendEmailVerification(user.email);
      }
    },
    
    // Before destroy
    beforeDestroy: async (user, options) => {
      // Archive user data
      await ArchiveUser.create({
        originalId: user.id,
        userData: user.toJSON()
      });
    },
    
    // After destroy
    afterDestroy: (user, options) => {
      console.log(`User ${user.id} has been deleted`);
    },
    
    // Before bulk operations
    beforeBulkCreate: (instances, options) => {
      console.log(`Creating ${instances.length} users`);
    },
    
    beforeBulkUpdate: (options) => {
      console.log('Bulk updating users');
    },
    
    beforeBulkDestroy: (options) => {
      console.log('Bulk deleting users');
    }
  }
});

// Adding hooks after model definition
User.addHook('beforeSave', 'hashPassword', async (user) => {
  if (user.changed('password')) {
    const bcrypt = require('bcrypt');
    user.password = await bcrypt.hash(user.password, 10);
  }
});

// Removing hooks
User.removeHook('beforeSave', 'hashPassword');

// Global hooks
sequelize.addHook('beforeDefine', (attributes, options) => {
  // Add timestamps to all models
  options.timestamps = true;
});
```

### Instance-Level Hooks

```javascript
const user = await User.create({
  firstName: 'John',
  lastName: 'Doe'
});

// Add hook to specific instance
user.addHook('afterSave', () => {
  console.log('This specific user was saved');
});
```

## CLI Usage

### Configuration

```javascript
// config/config.json
{
  "development": {
    "username": "root",
    "password": null,
    "database": "myapp_development",
    "host": "127.0.0.1",
    "dialect": "postgres",
    "logging": true
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "myapp_test",
    "host": "127.0.0.1",
    "dialect": "postgres",
    "logging": false
  },
  "production": {
    "use_env_variable": "DATABASE_URL",
    "dialect": "postgres",
    "dialectOptions": {
      "ssl": {
        "require": true,
        "rejectUnauthorized": false
      }
    }
  }
}
```

### Common CLI Commands

```bash
# Initialize project
npx sequelize-cli init

# Create database
npx sequelize-cli db:create

# Drop database
npx sequelize-cli db:drop

# Generate model
npx sequelize-cli model:generate --name User --attributes firstName:string,lastName:string,email:string,age:integer

# Generate migration
npx sequelize-cli migration:generate --name add-phone-to-users

# Run migrations
npx sequelize-cli db:migrate

# Undo migrations
npx sequelize-cli db:migrate:undo
npx sequelize-cli db:migrate:undo:all

# Migration status
npx sequelize-cli db:migrate:status

# Generate seeder
npx sequelize-cli seed:generate --name demo-users

# Run seeders
npx sequelize-cli db:seed:all
npx sequelize-cli db:seed --seed 20231201000000-demo-users.js

# Undo seeders
npx sequelize-cli db:seed:undo:all
```

### Environment-Specific Operations

```bash
# Specify environment
NODE_ENV=production npx sequelize-cli db:migrate

# Use different config file
npx sequelize-cli db:migrate --config config/database.json

# Use different models path
npx sequelize-cli db:migrate --models-path models/
```

## Best Practices

### Project Structure

```
project/
├── config/
│   ├── config.json          # Database configuration
│   └── database.js          # Database connection
├── models/
│   ├── index.js            # Auto-generated model loader
│   ├── user.js             # User model
│   └── post.js             # Post model
├── migrations/
│   ├── 20231201000000-create-users.js
│   └── 20231202000000-create-posts.js
├── seeders/
│   ├── 20231201000000-demo-users.js
│   └── 20231202000000-demo-posts.js
├── services/
│   ├── userService.js      # Business logic
│   └── postService.js
├── controllers/
│   ├── userController.js   # Route handlers
│   └── postController.js
└── app.js                  # Main application file
```

### Coding Best Practices

#### 1. Use Proper Error Handling

```javascript
// Good
try {
  const user = await User.findByPk(userId);
  if (!user) {
    throw new Error('User not found');
  }
  return user;
} catch (error) {
  console.error('Error finding user:', error);
  throw error;
}

// Better with custom errors
class UserNotFoundError extends Error {
  constructor(userId) {
    super(`User with ID ${userId} not found`);
    this.name = 'UserNotFoundError';
    this.statusCode = 404;
  }
}
```

#### 2. Use Transactions for Related Operations

```javascript
// Good
const createUserWithProfile = async (userData, profileData) => {
  return await sequelize.transaction(async (t) => {
    const user = await User.create(userData, { transaction: t });
    const profile = await Profile.create({
      ...profileData,
      userId: user.id
    }, { transaction: t });
    
    return { user, profile };
  });
};
```

#### 3. Implement Proper Validation

```javascript
// Model validation
const User = sequelize.define('User', {
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true,
    validate: {
      isEmail: true,
      notEmpty: true
    }
  }
}, {
  validate: {
    async emailNotExists() {
      if (this.changed('email') || this.isNewRecord) {
        const existing = await User.findOne({
          where: { 
            email: this.email,
            id: { [Op.ne]: this.id || 0 }
          }
        });
        if (existing) {
          throw new Error('Email already exists');
        }
      }
    }
  }
});
```

#### 4. Use Indexes Wisely

```javascript
// Add indexes for frequently queried columns
const User = sequelize.define('User', {
  email: {
    type: DataTypes.STRING,
    allowNull: false,
    unique: true // Creates unique index
  },
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,
  departmentId: DataTypes.INTEGER
}, {
  indexes: [
    // Single column index
    { fields: ['departmentId'] },
    
    // Composite index
    { fields: ['firstName', 'lastName'] },
    
    // Partial index
    {
      fields: ['email'],
      where: { isActive: true }
    },
    
    // Named index
    {
      name: 'user_name_search',
      fields: ['firstName', 'lastName']
    }
  ]
});
```

#### 5. Optimize Queries

```javascript
// Avoid N+1 queries
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts'
  }]
});

// Use separate queries for large datasets
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    separate: true
  }]
});

// Select only needed attributes
const users = await User.findAll({
  attributes: ['id', 'firstName', 'lastName', 'email']
});

// Use raw queries for complex operations
const result = await sequelize.query(`
  SELECT u.*, COUNT(p.id) as postCount
  FROM users u
  LEFT JOIN posts p ON u.id = p.authorId
  GROUP BY u.id
`, { type: QueryTypes.SELECT });
```

#### 6. Environment-Specific Configuration

```javascript
// config/database.js
const config = {
  development: {
    username: process.env.DB_USER || 'root',
    password: process.env.DB_PASS || '',
    database: process.env.DB_NAME || 'myapp_development',
    host: process.env.DB_HOST || 'localhost',
    dialect: 'postgres',
    logging: console.log
  },
  
  test: {
    username: process.env.DB_USER || 'root',
    password: process.env.DB_PASS || '',
    database: process.env.DB_NAME || 'myapp_test',
    host: process.env.DB_HOST || 'localhost',
    dialect: 'postgres',
    logging: false
  },
  
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'postgres',
    logging: false,
    pool: {
      max: 20,
      min: 0,
      acquire: 30000,
      idle: 10000
    },
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false
      }
    }
  }
};

module.exports = config[process.env.NODE_ENV || 'development'];
```

## Performance Optimization

### Connection Pooling

```javascript
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres',
  pool: {
    max: 20,        // Maximum connections
    min: 0,         // Minimum connections
    acquire: 60000, // Maximum time to get connection (ms)
    idle: 10000,    // Maximum idle time (ms)
    evict: 1000     // Check for idle connections interval
  }
});
```

### Query Optimization

```javascript
// Use indexes effectively
const users = await User.findAll({
  where: {
    departmentId: 1,    // Should have index
    isActive: true      // Should have index
  }
});

// Limit returned data
const users = await User.findAll({
  attributes: ['id', 'firstName', 'email'], // Only needed columns
  limit: 100,                               // Pagination
  offset: 0
});

// Use raw queries for complex operations
const stats = await sequelize.query(`
  SELECT 
    department_id,
    COUNT(*) as user_count,
    AVG(age) as avg_age
  FROM users 
  WHERE is_active = true 
  GROUP BY department_id
`, { type: QueryTypes.SELECT });
```

### Eager Loading Optimization

```javascript
// Avoid N+1 with proper includes
const users = await User.findAll({
  include: [{
    model: Department,
    as: 'department',
    attributes: ['id', 'name'] // Limit attributes
  }]
});

// Use separate queries for has-many relationships
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    separate: true,  // Separate query to avoid duplicates
    limit: 5         // Limit related records
  }]
});
```

### Bulk Operations

```javascript
// Bulk create
const users = await User.bulkCreate([
  { firstName: 'John', email: 'john@example.com' },
  { firstName: 'Jane', email: 'jane@example.com' }
], {
  validate: true,      // Validate each record
  individualHooks: false, // Skip individual hooks for performance
  updateOnDuplicate: ['firstName'] // Update on conflict
});

// Bulk update
await User.update(
  { isActive: false },
  {
    where: { lastLoginAt: { [Op.lt]: new Date('2023-01-01') } },
    individualHooks: false
  }
);
```

### Caching Strategies

```javascript
const Redis = require('redis');
const client = Redis.createClient();

// Cache frequently accessed data
const getCachedUser = async (userId) => {
  const cacheKey = `user:${userId}`;
  
  // Try cache first
  const cached = await client.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Fetch from database
  const user = await User.findByPk(userId);
  if (user) {
    // Cache for 1 hour
    await client.setex(cacheKey, 3600, JSON.stringify(user));
  }
  
  return user;
};

// Invalidate cache on updates
User.addHook('afterUpdate', async (user) => {
  await client.del(`user:${user.id}`);
});
```

## Testing

### Test Setup

```javascript
// test/setup.js
const { Sequelize } = require('sequelize');

// Use test database
const sequelize = new Sequelize('myapp_test', 'username', 'password', {
  host: 'localhost',
  dialect: 'postgres',
  logging: false
});

// Setup before tests
beforeAll(async () => {
  await sequelize.authenticate();
  await sequelize.sync({ force: true }); // Reset database
});

// Cleanup after tests
afterAll(async () => {
  await sequelize.close();
});

// Clean between tests
beforeEach(async () => {
  await sequelize.truncate({ cascade: true });
});

module.exports = { sequelize };
```

### Model Testing

```javascript
// test/models/user.test.js
const { User } = require('../../models');

describe('User Model', () => {
  describe('Validations', () => {
    test('should validate required fields', async () => {
      const user = User.build({});
      
      await expect(user.validate()).rejects.toThrow();
    });
    
    test('should validate email format', async () => {
      const user = User.build({
        firstName: 'John',
        lastName: 'Doe',
        email: 'invalid-email'
      });
      
      await expect(user.validate()).rejects.toThrow('Validation isEmail on email failed');
    });
    
    test('should create user with valid data', async () => {
      const userData = {
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com'
      };
      
      const user = await User.create(userData);
      
      expect(user.firstName).toBe('John');
      expect(user.email).toBe('john@example.com');
      expect(user.id).toBeDefined();
    });
  });
  
  describe('Associations', () => {
    test('should create user with profile', async () => {
      const user = await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com',
        Profile: {
          bio: 'Software Developer'
        }
      }, {
        include: [{ model: Profile, as: 'profile' }]
      });
      
      expect(user.Profile).toBeDefined();
      expect(user.Profile.bio).toBe('Software Developer');
    });
  });
  
  describe('Instance Methods', () => {
    test('should return full name', async () => {
      const user = await User.create({
        firstName: 'John',
        lastName: 'Doe',
        email: 'john@example.com'
      });
      
      expect(user.getFullName()).toBe('John Doe');
    });
  });
});
```

### Integration Testing

```javascript
// test/integration/userService.test.js
const { User, Profile } = require('../../models');
const UserService = require('../../services/userService');

describe('UserService Integration', () => {
  test('should create user with profile in transaction', async () => {
    const userData = {
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com'
    };
    
    const profileData = {
      bio: 'Software Developer',
      avatar: 'avatar.jpg'
    };
    
    const result = await UserService.createUserWithProfile(userData, profileData);
    
    expect(result.user.firstName).toBe('John');
    expect(result.profile.bio).toBe('Software Developer');
    expect(result.profile.userId).toBe(result.user.id);
    
    // Verify in database
    const dbUser = await User.findByPk(result.user.id, {
      include: [{ model: Profile, as: 'profile' }]
    });
    
    expect(dbUser.Profile.bio).toBe('Software Developer');
  });
});
```

### Testing Hooks

```javascript
// test/models/hooks.test.js
describe('User Hooks', () => {
  test('should hash password before create', async () => {
    const user = await User.create({
      firstName: 'John',
      lastName: 'Doe',
      email: 'john@example.com',
      password: 'plaintext'
    });
    
    expect(user.password).not.toBe('plaintext');
    expect(user.password).toMatch(/^\$2[ayb]\$[0-9]{2}\$/); // bcrypt format
  });
  
  test('should lowercase email before validation', async () => {
    const user = await User.create({
      firstName: 'John',
      lastName: 'Doe',
      email: 'JOHN@EXAMPLE.COM'
    });
    
    expect(user.email).toBe('john@example.com');
  });
});
```

## Common Patterns

### Service Layer Pattern

```javascript
// services/userService.js
const { User, Profile } = require('../models');
const { Op } = require('sequelize');

class UserService {
  static async createUser(userData) {
    try {
      const user = await User.create(userData);
      return { success: true, user };
    } catch (error) {
      if (error.name === 'SequelizeValidationError') {
        return { 
          success: false, 
          errors: error.errors.map(e => ({ field: e.path, message: e.message }))
        };
      }
      throw error;
    }
  }
  
  static async getUserById(id) {
    const user = await User.findByPk(id, {
      include: [{ model: Profile, as: 'profile' }]
    });
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return user;
  }
  
  static async updateUser(id, updates) {
    const user = await User.findByPk(id);
    if (!user) {
      throw new Error('User not found');
    }
    
    return await user.update(updates);
  }
  
  static async searchUsers(query, options = {}) {
    const { page = 1, limit = 10, sortBy = 'createdAt', sortOrder = 'DESC' } = options;
    
    const whereClause = query ? {
      [Op.or]: [
        { firstName: { [Op.iLike]: `%${query}%` } },
        { lastName: { [Op.iLike]: `%${query}%` } },
        { email: { [Op.iLike]: `%${query}%` } }
      ]
    } : {};
    
    return await User.findAndCountAll({
      where: whereClause,
      order: [[sortBy, sortOrder]],
      limit,
      offset: (page - 1) * limit,
      include: [{ model: Profile, as: 'profile' }]
    });
  }
  
  static async softDeleteUser(id) {
    const user = await User.findByPk(id);
    if (!user) {
      throw new Error('User not found');
    }
    
    return await user.destroy(); // Soft delete if paranoid: true
  }
}

module.exports = UserService;
```

### Repository Pattern

```javascript
// repositories/userRepository.js
class UserRepository {
  constructor(model) {
    this.model = model;
  }
  
  async create(data) {
    return await this.model.create(data);
  }
  
  async findById(id, options = {}) {
    return await this.model.findByPk(id, options);
  }
  
  async findAll(options = {}) {
    return await this.model.findAll(options);
  }
  
  async update(id, data) {
    const [updatedRowsCount] = await this.model.update(data, {
      where: { id }
    });
    
    if (updatedRowsCount === 0) {
      throw new Error('User not found');
    }
    
    return await this.findById(id);
  }
  
  async delete(id) {
    const deletedRowsCount = await this.model.destroy({
      where: { id }
    });
    
    return deletedRowsCount > 0;
  }
  
  async findByEmail(email) {
    return await this.model.findOne({
      where: { email }
    });
  }
  
  async findActiveUsers() {
    return await this.model.findAll({
      where: { isActive: true }
    });
  }
}

module.exports = UserRepository;

// Usage
const { User } = require('../models');
const userRepository = new UserRepository(User);
```

### Data Access Object (DAO) Pattern

```javascript
// dao/userDAO.js
const { User, Profile, sequelize } = require('../models');
const { Op } = require('sequelize');

class UserDAO {
  async createUserWithProfile(userData, profileData) {
    return await sequelize.transaction(async (t) => {
      const user = await User.create(userData, { transaction: t });
      const profile = await Profile.create({
        ...profileData,
        userId: user.id
      }, { transaction: t });
      
      return { user, profile };
    });
  }
  
  async getUsersWithPostCount() {
    return await User.findAll({
      attributes: [
        'id',
        'firstName',
        'lastName',
        'email',
        [sequelize.fn('COUNT', sequelize.col('Posts.id')), 'postCount']
      ],
      include: [{
        model: Post,
        as: 'posts',
        attributes: []
      }],
      group: ['User.id'],
      raw: true
    });
  }
  
  async getTopActiveUsers(limit = 10) {
    return await sequelize.query(`
      SELECT u.*, 
             COUNT(p.id) as post_count,
             MAX(p.created_at) as last_post_date
      FROM users u
      LEFT JOIN posts p ON u.id = p.author_id
      WHERE u.is_active = true
      GROUP BY u.id
      ORDER BY post_count DESC, last_post_date DESC
      LIMIT :limit
    `, {
      replacements: { limit },
      type: sequelize.QueryTypes.SELECT
    });
  }
}

module.exports = new UserDAO();
```

### Factory Pattern for Test Data

```javascript
// test/factories/userFactory.js
const { User } = require('../../models');
const faker = require('faker');

class UserFactory {
  static async create(overrides = {}) {
    const userData = {
      firstName: faker.name.firstName(),
      lastName: faker.name.lastName(),
      email: faker.internet.email(),
      age: faker.datatype.number({ min: 18, max: 65 }),
      isActive: true,
      ...overrides
    };
    
    return await User.create(userData);
  }
  
  static async createMany(count, overrides = {}) {
    const users = [];
    for (let i = 0; i < count; i++) {
      users.push(await this.create(overrides));
    }
    return users;
  }
  
  static build(overrides = {}) {
    return {
      firstName: faker.name.firstName(),
      lastName: faker.name.lastName(),
      email: faker.internet.email(),
      age: faker.datatype.number({ min: 18, max: 65 }),
      isActive: true,
      ...overrides
    };
  }
}

module.exports = UserFactory;

// Usage in tests
const user = await UserFactory.create({ email: 'test@example.com' });
const users = await UserFactory.createMany(5, { isActive: false });
```

## Troubleshooting

### Common Errors and Solutions

#### 1. Connection Issues

```javascript
// Error: connect ECONNREFUSED
// Solution: Check database connection details
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  port: 5432,
  dialect: 'postgres',
  retry: {
    max: 3
  }
});

// Test connection
sequelize.authenticate()
  .then(() => console.log('Connected'))
  .catch(err => console.error('Connection failed:', err));
```

#### 2. Migration Issues

```bash
# Error: Migration file already exists
# Solution: Check existing migrations
npx sequelize-cli db:migrate:status

# Error: Relation does not exist
# Solution: Run migrations in correct order
npx sequelize-cli db:migrate

# Rollback and retry
npx sequelize-cli db:migrate:undo
npx sequelize-cli db:migrate
```

#### 3. Association Issues

```javascript
// Error: Cannot read property 'id' of undefined
// Solution: Check if models are properly loaded and associated

// In models/index.js, ensure associations are set up after all models are loaded
Object.keys(db).forEach(modelName => {
  if (db[modelName].associate) {
    db[modelName].associate(db);
  }
});
```

#### 4. Validation Errors

```javascript
// Handle validation errors properly
try {
  await User.create(invalidData);
} catch (error) {
  if (error.name === 'SequelizeValidationError') {
    error.errors.forEach(err => {
      console.log(`${err.path}: ${err.message}`);
    });
  } else if (error.name === 'SequelizeUniqueConstraintError') {
    console.log('Unique constraint violation:', error.errors);
  } else {
    console.log('Other error:', error);
  }
}
```

#### 5. Memory Leaks

```javascript
// Always close connections
process.on('SIGINT', async () => {
  console.log('Closing database connection...');
  await sequelize.close();
  process.exit(0);
});

// Use connection pooling
const sequelize = new Sequelize('database', 'username', 'password', {
  pool: {
    max: 10,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
});
```

### Debug Techniques

#### Enable Logging

```javascript
const sequelize = new Sequelize('database', 'username', 'password', {
  logging: (sql, timing) => {
    console.log(`[${timing}ms] ${sql}`);
  },
  benchmark: true
});

// Or use custom logger
const winston = require('winston');
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [new winston.transports.File({ filename: 'sequelize.log' })]
});

const sequelize = new Sequelize('database', 'username', 'password', {
  logging: (msg) => logger.info(msg),
  benchmark: true
});
```

#### Query Analysis

```javascript
// Analyze query performance
const users = await User.findAll({
  where: { isActive: true },
  benchmark: true,
  logging: (sql, timing) => {
    if (timing > 1000) { // Log slow queries
      console.warn(`Slow query (${timing}ms): ${sql}`);
    }
  }
});

// Use EXPLAIN for complex queries
const [results, metadata] = await sequelize.query(
  'EXPLAIN ANALYZE SELECT * FROM users WHERE is_active = true',
  { type: sequelize.QueryTypes.SELECT }
);
```

#### Connection Pool Monitoring

```javascript
// Monitor connection pool
setInterval(() => {
  const pool = sequelize.connectionManager.pool;
  console.log(`Pool status:`, {
    size: pool.size,
    available: pool.available,
    using: pool.using,
    waiting: pool.waiting
  });
}, 10000);
```

### Performance Debugging

#### Identify N+1 Queries

```javascript
// Bad: N+1 query problem
const users = await User.findAll();
for (const user of users) {
  const posts = await user.getPosts(); // N queries
  console.log(`${user.name} has ${posts.length} posts`);
}

// Good: Use eager loading
const users = await User.findAll({
  include: [{ model: Post, as: 'posts' }]
});
users.forEach(user => {
  console.log(`${user.name} has ${user.posts.length} posts`);
});
```

#### Query Optimization Tools

```javascript
// Use query profiling
const { performance } = require('perf_hooks');

const profileQuery = async (queryFn) => {
  const start = performance.now();
  const result = await queryFn();
  const end = performance.now();
  console.log(`Query took ${end - start} milliseconds`);
  return result;
};

// Usage
const users = await profileQuery(() => 
  User.findAll({ include: [Post] })
);
```

## Advanced Topics

### Custom Data Types

```javascript
// Define custom data type
const { DataTypes } = require('sequelize');

const COORDINATES = {
  type: DataTypes.STRING,
  get() {
    const value = this.getDataValue('coordinates');
    if (!value) return null;
    
    const [lat, lng] = value.split(',').map(Number);
    return { latitude: lat, longitude: lng };
  },
  set(value) {
    if (!value || typeof value !== 'object') {
      this.setDataValue('coordinates', null);
      return;
    }
    
    this.setDataValue('coordinates', `${value.latitude},${value.longitude}`);
  }
};

const Location = sequelize.define('Location', {
  name: DataTypes.STRING,
  coordinates: COORDINATES
});

// Usage
const location = await Location.create({
  name: 'Central Park',
  coordinates: { latitude: 40.785091, longitude: -73.968285 }
});

console.log(location.coordinates); // { latitude: 40.785091, longitude: -73.968285 }
```

### Database Functions and Expressions

```javascript
// Using database functions
const users = await User.findAll({
  attributes: [
    'firstName',
    'lastName',
    [sequelize.fn('UPPER', sequelize.col('email')), 'upperEmail'],
    [sequelize.fn('CONCAT', sequelize.col('firstName'), ' ', sequelize.col('lastName')), 'fullName'],
    [sequelize.fn('DATE_PART', 'year', sequelize.col('createdAt')), 'yearCreated']
  ]
});

// Complex expressions
const userStats = await User.findAll({
  attributes: [
    'departmentId',
    [sequelize.fn('COUNT', sequelize.col('id')), 'userCount'],
    [sequelize.fn('AVG', sequelize.col('age')), 'averageAge'],
    [sequelize.literal('COUNT(*) FILTER (WHERE is_active = true)'), 'activeCount']
  ],
  group: ['departmentId']
});
```

### Window Functions

```javascript
// Using window functions
const usersWithRank = await sequelize.query(`
  SELECT 
    id,
    first_name,
    last_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) as salary_rank,
    ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as dept_rank
  FROM users
  WHERE is_active = true
`, { type: sequelize.QueryTypes.SELECT });
```

### Dynamic Model Creation

```javascript
// Create models dynamically
const createDynamicModel = (tableName, attributes) => {
  return sequelize.define(tableName, attributes, {
    tableName: tableName.toLowerCase(),
    timestamps: true
  });
};

// Usage
const CustomerModel = createDynamicModel('Customer', {
  name: DataTypes.STRING,
  email: DataTypes.STRING,
  phone: DataTypes.STRING
});

await CustomerModel.sync();
```

### Database Triggers Integration

```javascript
// Migration to create trigger
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Create trigger function
    await queryInterface.sequelize.query(`
      CREATE OR REPLACE FUNCTION update_modified_column()
      RETURNS TRIGGER AS $
      BEGIN
        NEW.updated_at = NOW();
        RETURN NEW;
      END;
      $ language 'plpgsql';
    `);
    
    // Create trigger
    await queryInterface.sequelize.query(`
      CREATE TRIGGER update_users_modtime
        BEFORE UPDATE ON users
        FOR EACH ROW
        EXECUTE FUNCTION update_modified_column();
    `);
  },
  
  down: async (queryInterface, Sequelize) => {
    await queryInterface.sequelize.query('DROP TRIGGER IF EXISTS update_users_modtime ON users');
    await queryInterface.sequelize.query('DROP FUNCTION IF EXISTS update_modified_column()');
  }
};
```

### Multi-Tenant Architecture

```javascript
// Schema-based multi-tenancy
const getTenantSequelize = (tenantId) => {
  return new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'postgres',
    schema: `tenant_${tenantId}`
  });
};

// Middleware for tenant selection
const tenantMiddleware = (req, res, next) => {
  const tenantId = req.headers['x-tenant-id'];
  if (!tenantId) {
    return res.status(400).json({ error: 'Tenant ID required' });
  }
  
  req.sequelize = getTenantSequelize(tenantId);
  next();
};

// Database-based multi-tenancy
const User = sequelize.define('User', {
  tenantId: {
    type: DataTypes.INTEGER,
    allowNull: false
  },
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING
}, {
  defaultScope: {
    where: {
      tenantId: 1 // Default tenant
    }
  },
  scopes: {
    tenant: (tenantId) => ({
      where: { tenantId }
    })
  }
});
```

### Conclusion

This comprehensive guide covers the essential aspects of Sequelize, from basic setup to advanced patterns. Key takeaways:

1. **Start with basics**: Understand models, associations, and migrations
2. **Use proper patterns**: Implement service layers and proper error handling
3. **Optimize performance**: Use indexes, proper queries, and connection pooling
4. **Test thoroughly**: Write comprehensive tests for models and business logic
5. **Monitor and debug**: Use logging and profiling tools
6. **Follow best practices**: Use transactions, validation, and proper project structure

### Additional Resources

- [Sequelize Documentation](https://sequelize.org/docs/v6/)
- [Sequelize GitHub Repository](https://github.com/sequelize/sequelize)
- [Sequelize CLI Documentation](https://sequelize.org/docs/v6/other-topics/migrations/)
- [Database Design Best Practices](https://www.postgresql.org/docs/current/ddl-best-practices.html)
- [Node.js Performance Best Practices](https://nodejs.org/en/docs/guides/simple-profiling/)

Remember to always refer to the official documentation for the most up-to-date information and best practices.

# MongoDB/Mongoose Complete Cheat Sheet

## Folder Structure:
```markdown
project/
├── server.js               # Entry point: starts server
├── package.json
└── src/
    ├── app.js              # Express app initialization
    ├── config/
    │   └── db.js           # MongoDB connection
    ├── models/             # Mongoose schemas & models
    │   ├── User.js
    │   └── Post.js
    ├── controllers/        # Business logic, interact with models
    │   ├── userController.js
    │   └── postController.js
    ├── routes/             # Express routes
    │   ├── userRoutes.js
    │   └── postRoutes.js
    ├── middlewares/        # Auth, logging, error handling
    └── utils/              # Helper functions
```
---
## Table of Contents
1. [Basic Setup](#basic-setup)
2. [Schema & Models](#schema--models)
3. [CRUD Operations](#crud-operations)
4. [Query Operations](#query-operations)
5. [Aggregation Pipeline](#aggregation-pipeline)
6. [Indexes](#indexes)
7. [Population](#population)
8. [Middleware](#middleware)
9. [Validation](#validation)
10. [Utilities](#utilities)

---

## Basic Setup

```javascript
const mongoose = require('mongoose');

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Connection events
mongoose.connection.on('connected', () => console.log('Connected to MongoDB'));
mongoose.connection.on('error', (err) => console.log('MongoDB error:', err));
mongoose.connection.on('disconnected', () => console.log('Disconnected from MongoDB'));
```

---

## Schema & Models

```javascript
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, unique: true, required: true },
  age: { type: Number, min: 0, max: 120 },
  isActive: { type: Boolean, default: true },
  tags: [String],
  profile: {
    bio: String,
    website: String
  },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
```

### Schema Types
```javascript
String, Number, Date, Buffer, Boolean, Mixed, ObjectId, Array, Decimal128, Map
```

---

## CRUD Operations

### Create
```javascript
// Single document
const user = new User({ name: 'John', email: 'john@email.com' });
await user.save();

// Or using create
const user = await User.create({ name: 'John', email: 'john@email.com' });

// Multiple documents
await User.insertMany([
  { name: 'Alice', email: 'alice@email.com' },
  { name: 'Bob', email: 'bob@email.com' }
]);
```

### Read
```javascript
// Find all
const users = await User.find();

// Find with conditions
const activeUsers = await User.find({ isActive: true });

// Find one
const user = await User.findOne({ email: 'john@email.com' });

// Find by ID
const user = await User.findById('507f1f77bcf86cd799439011');

// Select specific fields
const users = await User.find().select('name email -_id');

// Limit and skip
const users = await User.find().limit(10).skip(20);
```

### Update
```javascript
// Update one document
await User.updateOne({ _id: userId }, { $set: { name: 'Updated Name' } });

// Update multiple documents
await User.updateMany({ isActive: false }, { $set: { isActive: true } });

// Find and update
const user = await User.findByIdAndUpdate(userId, 
  { name: 'New Name' }, 
  { new: true, runValidators: true }
);

// Find one and update
const user = await User.findOneAndUpdate(
  { email: 'old@email.com' },
  { email: 'new@email.com' },
  { new: true }
);
```

### Delete
```javascript
// Delete one document
await User.deleteOne({ _id: userId });

// Delete multiple documents
await User.deleteMany({ isActive: false });

// Find and delete
const deletedUser = await User.findByIdAndDelete(userId);

// Find one and delete
const deletedUser = await User.findOneAndDelete({ email: 'delete@email.com' });
```

---

## Query Operations

### Comparison Operators
```javascript
// $eq - Equal
await User.find({ age: { $eq: 25 } });

// $ne - Not equal
await User.find({ age: { $ne: 25 } });

// $gt - Greater than
await User.find({ age: { $gt: 18 } });

// $gte - Greater than or equal
await User.find({ age: { $gte: 18 } });

// $lt - Less than
await User.find({ age: { $lt: 65 } });

// $lte - Less than or equal
await User.find({ age: { $lte: 65 } });

// $in - In array
await User.find({ age: { $in: [20, 25, 30] } });

// $nin - Not in array
await User.find({ age: { $nin: [20, 25, 30] } });
```

### Logical Operators
```javascript
// $and
await User.find({ $and: [{ age: { $gte: 18 } }, { age: { $lt: 65 } }] });

// $or
await User.find({ $or: [{ name: 'John' }, { name: 'Jane' }] });

// $not
await User.find({ age: { $not: { $gt: 65 } } });

// $nor - Not or
await User.find({ $nor: [{ name: 'John' }, { age: 25 }] });
```

### Element Operators
```javascript
// $exists - Field exists
await User.find({ email: { $exists: true } });

// $type - Field type
await User.find({ age: { $type: 'number' } });
```

### Array Operators
```javascript
// $all - All elements match
await User.find({ tags: { $all: ['javascript', 'mongodb'] } });

// $elemMatch - Element match
await User.find({ 
  scores: { $elemMatch: { $gte: 80, $lt: 85 } }
});

// $size - Array size
await User.find({ tags: { $size: 3 } });
```

### Text Search
```javascript
// Create text index first
userSchema.index({ name: 'text', 'profile.bio': 'text' });

// Text search
await User.find({ $text: { $search: 'john developer' } });
```

### Regular Expressions
```javascript
// Case insensitive search
await User.find({ name: /john/i });

// Starts with
await User.find({ name: /^john/i });

// Ends with
await User.find({ email: /gmail\.com$/ });
```

---

## Aggregation Pipeline

### Basic Aggregation
```javascript
const result = await User.aggregate([
  { $match: { isActive: true } },
  { $group: { _id: '$department', count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);
```

### $match - Filter documents
```javascript
// Match active users
{ $match: { isActive: true } }

// Match with multiple conditions
{ $match: { $and: [{ age: { $gte: 18 } }, { isActive: true }] } }
```

### $group - Group documents
```javascript
// Group by field and count
{ $group: { _id: '$department', count: { $sum: 1 } } }

// Group with multiple accumulators
{
  $group: {
    _id: '$department',
    totalUsers: { $sum: 1 },
    averageAge: { $avg: '$age' },
    maxSalary: { $max: '$salary' },
    minSalary: { $min: '$salary' },
    userNames: { $push: '$name' },
    uniqueSkills: { $addToSet: '$skills' }
  }
}
```

### $project - Select and reshape fields
```javascript
// Include/exclude fields
{ $project: { name: 1, email: 1, _id: 0 } }

// Create computed fields
{
  $project: {
    name: 1,
    email: 1,
    fullName: { $concat: ['$firstName', ' ', '$lastName'] },
    ageGroup: {
      $cond: {
        if: { $gte: ['$age', 18] },
        then: 'Adult',
        else: 'Minor'
      }
    }
  }
}
```

### $sort - Sort documents
```javascript
// Sort by single field
{ $sort: { createdAt: -1 } } // -1 for descending, 1 for ascending

// Sort by multiple fields
{ $sort: { department: 1, salary: -1 } }
```

### $limit and $skip - Pagination
```javascript
{ $skip: 20 }   // Skip first 20 documents
{ $limit: 10 }  // Limit to 10 documents
```

### $lookup - Join collections
```javascript
// Basic lookup
{
  $lookup: {
    from: 'posts',           // Collection to join
    localField: '_id',       // Field from input documents
    foreignField: 'userId',  // Field from documents of 'from' collection
    as: 'userPosts'         // Output array field
  }
}

// Lookup with pipeline (more complex)
{
  $lookup: {
    from: 'posts',
    let: { userId: '$_id' },
    pipeline: [
      { $match: { $expr: { $eq: ['$userId', '$$userId'] } } },
      { $match: { published: true } },
      { $sort: { createdAt: -1 } },
      { $limit: 5 }
    ],
    as: 'recentPosts'
  }
}
```

### $unwind - Deconstruct arrays
```javascript
// Basic unwind
{ $unwind: '$tags' }

// Unwind with options
{
  $unwind: {
    path: '$tags',
    includeArrayIndex: 'tagIndex',    // Add index field
    preserveNullAndEmptyArrays: true  // Keep docs with empty arrays
  }
}
```

### $addFields - Add new fields
```javascript
{
  $addFields: {
    fullName: { $concat: ['$firstName', ' ', '$lastName'] },
    isAdult: { $gte: ['$age', 18] },
    tagsCount: { $size: '$tags' }
  }
}
```

### $replaceRoot - Replace document root
```javascript
// Replace with subdocument
{ $replaceRoot: { newRoot: '$profile' } }

// Replace with computed document
{
  $replaceRoot: {
    newRoot: {
      name: '$name',
      contact: {
        email: '$email',
        phone: '$phone'
      }
    }
  }
}
```

### $facet - Multi-faceted aggregation
```javascript
{
  $facet: {
    'ageGroups': [
      { $group: { _id: { $floor: { $divide: ['$age', 10] } }, count: { $sum: 1 } } }
    ],
    'departments': [
      { $group: { _id: '$department', count: { $sum: 1 } } }
    ],
    'totalCount': [
      { $count: 'total' }
    ]
  }
}
```

### $bucket - Group by ranges
```javascript
{
  $bucket: {
    groupBy: '$age',
    boundaries: [0, 18, 30, 50, 100],
    default: 'Other',
    output: {
      count: { $sum: 1 },
      averageSalary: { $avg: '$salary' }
    }
  }
}
```

### $sample - Random sample
```javascript
{ $sample: { size: 5 } } // Get 5 random documents
```

### $count - Count documents
```javascript
{ $count: 'totalUsers' }
```

### $out and $merge - Output to collection
```javascript
// Replace collection
{ $out: 'userStats' }

// Merge with existing collection
{
  $merge: {
    into: 'userStats',
    whenMatched: 'merge',
    whenNotMatched: 'insert'
  }
}
```

---

## Indexes

### Create Indexes
```javascript
// Single field index
await User.createIndex({ email: 1 }); // 1 for ascending, -1 for descending

// Compound index
await User.createIndex({ department: 1, salary: -1 });

// Text index
await User.createIndex({ name: 'text', bio: 'text' });

// Partial index
await User.createIndex(
  { email: 1 },
  { partialFilterExpression: { isActive: true } }
);

// TTL index (expires documents)
await User.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Unique index
await User.createIndex({ email: 1 }, { unique: true });

// Sparse index
await User.createIndex({ phone: 1 }, { sparse: true });
```

### Schema-level Indexes
```javascript
const userSchema = new mongoose.Schema({
  email: { type: String, index: true, unique: true },
  name: { type: String, index: true },
  createdAt: { type: Date, index: true }
});

// Compound index in schema
userSchema.index({ department: 1, salary: -1 });

// Text index in schema
userSchema.index({ name: 'text', bio: 'text' });
```

### Index Management
```javascript
// List indexes
await User.listIndexes();

// Drop index
await User.dropIndex({ email: 1 });

// Drop all indexes
await User.dropIndexes();
```

---

## Population

### Basic Population
```javascript
const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: { type: mongoose.Schema.Types.ObjectId, ref: 'User' }
});

const Post = mongoose.model('Post', postSchema);

// Populate author
const posts = await Post.find().populate('author');

// Populate specific fields
const posts = await Post.find().populate('author', 'name email');

// Populate with conditions
const posts = await Post.find().populate({
  path: 'author',
  match: { isActive: true },
  select: 'name email'
});
```

### Nested Population
```javascript
const posts = await Post.find().populate({
  path: 'author',
  populate: {
    path: 'company',
    model: 'Company'
  }
});
```

### Virtual Population
```javascript
userSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'author'
});

// Enable virtual fields in JSON
userSchema.set('toJSON', { virtuals: true });

const user = await User.findById(userId).populate('posts');
```

---

## Middleware

### Pre Middleware
```javascript
// Pre save
userSchema.pre('save', function(next) {
  if (this.isModified('password')) {
    this.password = hashPassword(this.password);
  }
  next();
});

// Pre find
userSchema.pre('find', function() {
  this.populate('company');
});

// Pre aggregate
userSchema.pre('aggregate', function() {
  this.pipeline().unshift({ $match: { isDeleted: { $ne: true } } });
});
```

### Post Middleware
```javascript
// Post save
userSchema.post('save', function(doc, next) {
  console.log('User saved:', doc._id);
  next();
});

// Post find
userSchema.post('find', function(docs) {
  console.log('Found', docs.length, 'documents');
});
```

---

## Validation

### Built-in Validators
```javascript
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Name is required'],
    minlength: [2, 'Name must be at least 2 characters'],
    maxlength: [50, 'Name cannot exceed 50 characters'],
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email']
  },
  age: {
    type: Number,
    min: [0, 'Age cannot be negative'],
    max: [120, 'Age cannot exceed 120']
  }
});
```

### Custom Validators
```javascript
userSchema.path('email').validate(async function(email) {
  const user = await mongoose.models.User.findOne({ email });
  return !user || user._id.equals(this._id);
}, 'Email already exists');

// Custom validator function
userSchema.path('phone').validate(function(phone) {
  return /^\d{10}$/.test(phone);
}, 'Phone must be 10 digits');
```

---

## Utilities

### Timestamps
```javascript
const schema = new mongoose.Schema({
  // ... fields
}, { timestamps: true }); // Adds createdAt and updatedAt
```

### Transform Output
```javascript
userSchema.methods.toJSON = function() {
  const user = this.toObject();
  delete user.password;
  return user;
};
```

### Instance Methods
```javascript
userSchema.methods.comparePassword = function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// Usage
const user = await User.findById(userId);
const isMatch = await user.comparePassword('plaintextPassword');
```

### Static Methods
```javascript
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// Usage
const user = await User.findByEmail('JOHN@EMAIL.COM');
```

### Query Helpers
```javascript
userSchema.query.byDepartment = function(department) {
  return this.where({ department: department });
};

// Usage
const users = await User.find().byDepartment('Engineering');
```

### Virtuals
```javascript
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

userSchema.virtual('fullName').set(function(name) {
  const parts = name.split(' ');
  this.firstName = parts[0];
  this.lastName = parts[1];
});
```

---

## Common Aggregation Patterns

### Calculate Statistics
```javascript
const stats = await User.aggregate([
  {
    $group: {
      _id: null,
      totalUsers: { $sum: 1 },
      averageAge: { $avg: '$age' },
      oldestUser: { $max: '$age' },
      youngestUser: { $min: '$age' }
    }
  }
]);
```

### Group by Date Ranges
```javascript
const monthlyStats = await User.aggregate([
  {
    $group: {
      _id: {
        year: { $year: '$createdAt' },
        month: { $month: '$createdAt' }
      },
      count: { $sum: 1 }
    }
  },
  { $sort: { '_id.year': 1, '_id.month': 1 } }
]);
```

### Top N Query
```javascript
const topUsers = await User.aggregate([
  { $match: { isActive: true } },
  { $sort: { score: -1 } },
  { $limit: 10 },
  { $project: { name: 1, score: 1 } }
]);
```

### Conditional Aggregation
```javascript
const userStats = await User.aggregate([
  {
    $group: {
      _id: '$department',
      totalUsers: { $sum: 1 },
      activeUsers: {
        $sum: { $cond: [{ $eq: ['$isActive', true] }, 1, 0] }
      },
      seniorUsers: {
        $sum: { $cond: [{ $gte: ['$age', 50] }, 1, 0] }
      }
    }
  }
]);
```

---

## Performance Tips

1. **Use indexes** for frequently queried fields
2. **Project early** in aggregation pipelines to reduce data size
3. **Use $match early** in pipelines to filter documents
4. **Avoid $lookup** on large collections when possible
5. **Use explain()** to analyze query performance
6. **Consider compound indexes** for multi-field queries
7. **Use lean()** for read-only operations to get plain JS objects
8. **Batch operations** using bulkWrite() for multiple updates

```javascript
// Example of explain
await User.find({ department: 'Engineering' }).explain('executionStats');

// Example of lean
const users = await User.find().lean(); // Returns plain JS objects

// Example of bulkWrite
await User.bulkWrite([
  { updateOne: { filter: { _id: id1 }, update: { $set: { isActive: false } } } },
  { updateOne: { filter: { _id: id2 }, update: { $set: { isActive: true } } } }
]);
```
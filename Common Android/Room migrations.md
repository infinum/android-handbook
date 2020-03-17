# Database migrations

Schema migrations, or database migrations, refer to the management of incremental changes of database schemas. A migration is performed on a database whenever it is necessary to update or revert that database's schema to some newer or older version. When a user updates to the latest version of your app, you don't want them to lose all of their existing data.

Applying a migration to a production database is always a risk. They are usually very large and full of surprises. The surprises can come from many sources:

- Corrupt data that was written by old versions of the software and not cleaned properly
- Implied dependencies in the data which no one knows about anymore
- Bugs in the schema migration tools
- Mistakes in assumptions how data should be migrated

### Should you write one?

So, your app has some sort of a database in it. Is it mandatory to write migrations for it, you wonder? Below listed are some of the most common cases when you should be defining migrations:

1. Adding columns (entity fields)
2. Changing indices
3. Fixing mistakes
4. Corrupted data
5. Modifying data types

# Making it work in Room

If you're wondering how things worked before Room, look [here](https://riggaroo.co.za/android-sqlite-database-use-onupgrade-correctly) to find out how it's done directly through the SQLite API.

### Step 1: Setting up build.gradle

Before you even get started writing queries, you should first set up a directory for your database schemas. These schemas are generated automagically whenever you change your database version.

```groovy
android {
    defaultConfig {
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation": "$projectDir/schemas".toString()]
            }
        }
    }
}
```

Additionally, you will have to add your newly defined folder to the assets list, so that it can be accessed from inside your tests. 

```groovy
android {
    sourceSets {
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}
```

### Step 2: Modify your entites, DAOs and DBs

You can now modify your existing Room entities, and DAOs. You're free to do whatever you want here, but be aware that you also must update the connected DB version in order to generate a successful build. Otherwise, Room is going to fail with an `IllegalStateException`, stating that:

> Room cannot verify the data integrity. Looks like you've changed schema but forgot to update the version number. You can simply fix this by increasing the version number.

### Step 3: Update the DB version

Obviously, you need to update the DB version. This is done by incrementing the version number by at least one. It's important that you **never decrease** this number, even if you're "reverting" to an older state of your database. Be careful, you're not done yet. If you run your app now, Room will fail again, but with a different message this time:

> A migration from <old version> to <new version> is necessary. Please provide a Migration in the builder or call fallbackToDestructiveMigration in the builder in which case Room will re-create all of the tables.

Even though it is mentioned as an option, **DO NOT** choose `fallbackToDestructiveMigration`. What this does is delete the entire database and all the data in it, and then building a new one based on the modified schema. This leads to complete data loss, which is never an option. What you **SHOULD** do is write a Migration step.

### Step 4: Writing the migration

So, it's finally time to write a migration query. In Room, a migration query is wrapped into a `Migration` class. In the constructor, you define the old version that you're migrating from and the new version you're migrating to. 

Additionally, you also must provide an implementation of the abstract method `migrate`. This is where you'll be writing your SQL statement.

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Begin SQL transcation
        database.execSQL("BEGIN TRANSACTION;")


        // Alter, create, drop, whatever
        database.execSQL("ALTER TABLE DemoName ADD COLUMN demo_column INTEGER;")

        // Drop the existing index
        database.execSQL("DROP INDEX IF EXISTS 'old_index_name';")
		
        // Create new index
        database.execSQL(
            """
                CREATE INDEX 'new_index_name'
                    ON 'DemoName' ('primary_key_column_one', 'primary_key_column_two');
            """.trimIndent()
        )

        // Commit transaction
        database.execSQL("COMMIT;")
    }
}
```

Once you're satisfied with your migration statement, add it to the proper Room database builder. It also wouldn't hurt to leave an inline comment that shortly describes the changes made in this migration:

```kotlin
fun syncDatabase(appContext: Context): SyncActionDb = Room.databaseBuilder(appContext, DemoDb::class.java, "demo-db")
    .addMigrations(
        MIGRATION_1_2 // Add demo_column to DemoName
    )
    .build()
```

You can now build and run your app without fear of loosing data. Or can you? It's important to note that the SQL statements you execute in the `migrate` method are **not checked at compile time**, meaning that they could fail without warning.

### Things to watch out for

Before we move on, let's go over the things that might kill your app one more time:

1. **Destructive fallbacks** = Don't use `fallbackToDestructiveMigration`. Never, ever, ever. Not even then.
2. **One command per `db.execSQL`** = `db.execSQL` only handles one SQL command per call, all others are ignored
3. **DB version number** = Don't forget to increment the DB version when you make changes to entities
4. **Indexes** = When writing migration queries, don't forget to update the indexes too

# Testing, testing, one, two

Migrations aren't trivial to write, and failure to write them properly could cause a crash in your app. Even more worrying, it could work just fine, but you could lose precious data in the migration process. To preserve your app's stability, you should test your migrations beforehand. 

### Dependencies

Room provides a testing Maven artifact to assist with this testing process (to find the latest version, look [here](https://developer.android.com/jetpack/androidx/releases/room)):

```groovy
androidTestImplementation “android.arch.persistence.room:testing:<room_version>”
```

Also, we've already mentioned in [step 1](#step-1-setting-up-build.gradle) that you need to define the schema location and add that location to the source sets in order for tests to function, so if you haven't done that, do it now.

### Setting it up

Before we begin writing our migration tests, first we need to set up the actual test class. Unlike most tests we write, migration tests have to be written as instrumentation tests. This means that they require a device to run, and should also be positioned in the `androidTest` folder inside your project. A crude setup of a single test class should look something like this:

```kotlin
@RunWith(AndroidJUnit4::class)
class DemoDbMigrationTest {
	
    companion object {
        private const val TEST_DEMO_DB_NAME = "test-demo-db"
    }

	@Rule
	var migrationTestHelper = MigrationTestHelper(
	    InstrumentationRegistry.getInstrumentation(),
	    DemoDb::class.java.canonicalName,
	    FrameworkSQLiteOpenHelperFactory()
	)
	
    @Test
    fun migrationTest() {
    	// Write a test here
    }
}
```

To properly run tests with databases, you need to be able to create, open and close them, as well as run the actual migrations. This ends up being a lot of boilerplate code, so Room offers the `MigrationTestHelper` to assist you in your troubles.

### Writing a test

It's finally time to write a migration test. A single migration requires at least two separate test cases: 

#### 1. Migrating from old to new DB version

```kotlin
@Test
@Throws(IOException::class)
fun migrationFrom1To2() {
    // Create the database with version 1
    val db = migrationTestHelper.createDatabase(TEST_DEMO_DB_NAME, 1)
	
    // Insert some data
    insertData(DEMO_DATA, db, 1)
	
    // Prepare for the next version
    db.close()
	
    // Re-open the database with version 2 and provide migrations to run
    migrationTestHelper.runMigrationsAndValidate(
        TEST_DEMO_DB_NAME,
        2,
        true,
        MIGRATION_1_2
    )

    // Get the migrated version of the database
    val migratedDb = getMigratedRoomDatabase()
    val data = migratedDb.dataDao().get(DEMO_DATA.demoId).blockingGet()
	
    // Verify that the data is correct
    assertThat(data.demoId).isEqualTo(DEMO_DATA.demoId)
    // The previous version did not contain this field so it couldn't have been set
    assertThat(data.demoField).isEqualTo(null)
}
```

Finally, the code block above is what you came here for. The test case begins by creating an old version of the database and inserting some data into it. Once complete, the database is migrated to the newly defined version, after which the migrated data is verified.


#### 2. Starting from newest version

```kotlin
@Test
@Throws(IOException::class)
fun startInVersion2() {
    // Create the database with version 2
    val db = migrationTestHelper.createDatabase(TEST_DEMO_DB_NAME, 2)

    // Insert some data
    insertData(DEMO_DATA, db)

    // Prepare for the next version
    db.close()

    // Get the latest, migrated, version of the database
    val migratedDb = getMigratedRoomDatabase()
    val data = migratedDb.dataDao().get(DEMO_DATA.demoId).blockingGet()

    // Verify that the data is correct
    assertThat(data.demoId).isEqualTo(DEMO_DATA.demoId)
    assertThat(data.demoField).isEqualTo(DEMO_DATA.demoField)
}
```

One more thing to do - test a fresh app install, which creates a new database matching the latest version. We don't test the actual migration here, just if the newest DB version behaves correctly. This time, the newest version of the database is created, data is inserted and the latest version is fetched again, with all migrations activated. The data is validated and the test is complete.

# Resources

For more info, check out some of these resources:

1. [Understanding migrations with Room](https://medium.com/androiddevelopers/understanding-migrations-with-room-f01e04b07929)
2. [Testing Room migrations](https://medium.com/androiddevelopers/testing-room-migrations-be93cdb0d975)
3. [Migrating Room databases](https://developer.android.com/training/data-storage/room/migrating-db-versions)
4. [Room migration sample project](https://github.com/android/architecture-components-samples/tree/master/PersistenceMigrationsSample)

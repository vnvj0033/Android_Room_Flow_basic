## Flow
```kotlin
class BusScheduleViewModel(private val scheduleDao: ScheduleDao): ViewModel() {
    fun fullSchedule(): Flow<List<Schedule>> = scheduleDao.getAll()
}

lifecycleScope.launch(Dispatchers.IO) {
    viewModel.fullSchedule().collect {
        busStopAdapter.submitList(it)
    }
}
```

## ViewModel define
```
class MyViewModel(val dao: MyDao): ViewModel()

// use viewmodel
private val viewModel: MyViewModel by viewModels()
```

## Room Database
```kotlin
@Database(entities = [Schedule::class], version = 1)
abstract class AppDatabase: RoomDatabase() {

    abstract fun scheduleDao(): ScheduleDao

    companion object {

        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance =
                    Room.databaseBuilder(context, AppDatabase::class.java, "app_database")
                        .createFromAsset("database/bus_schedule.db")
                        .build()

                INSTANCE = instance
                instance
            }
        }
    }
}

// use database
val database: AppDatabase by lazy { AppDatabase.getDatabase(this) }
```

## Room DAO
Flow를 사용하기 위해 return 타입을 Flow로 포장
```kotlin
@Dao
interface ScheduleDao {
    @Query("SELECT * FROM schedule ORDER BY arrival_time ASC")
    fun getAll(): Flow<List<Schedule>> 

    @Query("SELECT * FROM schedule WHERE stop_name = :stopName ORDER BY arrival_time ASC")
    fun getByStopName(stopName: String): Flow<List<Schedule>>
}
```

## Room entry
```kotlin
@Entity
data class Schedule(
   @PrimaryKey val id: Int,
   @NonNull @ColumnInfo(name = "stop_name") val stopName: String,
   @NonNull @ColumnInfo(name = "arrival_time") val arrivalTime: Int
)
```

# Bus Scheduler App

This folder contains the source code for the Bus Scheduler app codelab.<br />
https://developer.android.com/codelabs/basic-android-kotlin-training-intro-room-flow

# Introduction
The Bus Scheduler app displays a list of bus stops and arrival times. Tapping a bus stop on the first screen will display a list of all arrival times for that particular stop.

The bus stops are stored in a Room database. Schedule items are represented by the `Schedule` class and queries on the data table are made by the `ScheduleDao` class. The app includes a view model to access the `ScheduleDao` and format data to be display in a list, using `Flow` to send data to a recycler view adapter.

# Pre-requisites
* Experience with Kotlin syntax.
* Familiarity with activities, fragments, and recycler views.
* Basic knowledge of SQL databases and performing basic queries.

# Getting Started
1. Install Android Studio, if you don't already have it.
2. Download the sample.
3. Import the sample into Android Studio.
4. Build and run the sample.

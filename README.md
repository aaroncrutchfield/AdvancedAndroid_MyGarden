# Exercise 2 - Add Watering Service

## _plant_widget.xml_
#### 1. Add an `ImageView` that displays `water_drop_blue` to act as the watering button. [[code][1]]
```xml
    <ImageView
        android:id="@+id/widget_water_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal"
        android:src="@drawable/water_drop_blue"/>
```


## _PlantWateringService.java_
#### 2. Create a plant watering service that extends `IntentService` and supports the action `ACTION_WATER_PLANTS` which updates `last_watered` timestamp for all plants still alive. [[code][2]]
```java
public class PlantWateringService extends IntentService {


    public static final String ACTION_WATER_PLANTS = "com.example.android.mygarden.action.water_plants";


    public PlantWateringService() {
        super(PlantWateringService.class.getSimpleName());
    }




    @Override
    protected void onHandleIntent(@Nullable Intent intent) {
        if (intent != null) {
            final String action = intent.getAction();
            if(ACTION_WATER_PLANTS.equals(action)) {
                handleActionWaterPlants();
            }
        }
    }


    /**
     * Handle action WaterPlant in the provided background thread with the provided
     * parameters.
     */
    private void handleActionWaterPlants() {
        Uri PLANTS_URI = PlantContract.BASE_CONTENT_URI
                .buildUpon()
                .appendPath(PlantContract.PATH_PLANTS)
                .build();
        ContentValues contentValues = new ContentValues();
        long timeNow = System.currentTimeMillis();
        contentValues.put(PlantEntry.COLUMN_LAST_WATERED_TIME, timeNow);
        // Update only plants that are still alive
        getContentResolver().update(
                PLANTS_URI,
                contentValues,
                PlantEntry.COLUMN_LAST_WATERED_TIME + ">?",
                new String\[\]{String.valueOf(timeNow - PlantUtils.MAX_AGE_WITHOUT_WATER)});
    }


    /**
     * Starts this service to perform WaterPlants action with the given parameters. If
     * the service is already performing a task this action will be queued.
     *
     * @see IntentService
     */
    public static void startActionWaterPlants(Context context) {
        Intent intent = new Intent(context, PlantWateringService.class);
        intent.setAction(ACTION_WATER_PLANTS);
        context.startService(intent);
    }
}
```



## _AndroidManifest.xml_
#### 3. Add a service tag for `PlantWateringService`. [[code][3]]
```xml
        <service android:name=".PlantWateringService"/>
```



## _PlantWidgetProvider.java_
#### 4. Create a `PendingIntent` for the `PlantWateringService` and `setOnClickPendingIntent` for `widget_water_button`. [[code][4]]
```java
        Intent wateringIntent = new Intent(context, PlantWateringService.class);
        wateringIntent.setAction(PlantWateringService.ACTION_WATER_PLANTS);
        PendingIntent wateringPendingIntent = PendingIntent.getService(
                context,
                0,
                wateringIntent,
                PendingIntent.FLAG_UPDATE_CURRENT);
        views.setOnClickPendingIntent(R.id.widget_water_button, wateringPendingIntent);
```



## _Screenshots_
![1](screenshots/1.gif)


[1]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/49246f7afe0d0475c9ff8b131dd1ba1ff2436a23/app/src/main/res/layout/plant_widget.xml#L25-L30
[2]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/49246f7afe0d0475c9ff8b131dd1ba1ff2436a23/app/src/main/java/com/example/android/mygarden/PlantWateringService.java#L18-L68
[3]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/49246f7afe0d0475c9ff8b131dd1ba1ff2436a23/app/src/main/AndroidManifest.xml#L24
[4]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/49246f7afe0d0475c9ff8b131dd1ba1ff2436a23/app/src/main/java/com/example/android/mygarden/PlantWidgetProvider.java#L42-L49
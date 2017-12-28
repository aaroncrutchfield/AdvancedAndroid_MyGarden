# Exercise 3 - Update Widgets Service

## _PlantWidgetProvider.java_
#### 1. Modify `updateAppWidget` method to take an image recourse and call `setImageViewResource` to update the widget’s image. [[code][1]]
```java
    static void updateAppWidget(Context context, AppWidgetManager appWidgetManager,
                                int imgRes, int appWidgetId) {


        // Create an Intent to launch MainActivity when clicked
        Intent intent = new Intent(context, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);
        // Construct the RemoteViews object
        RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.plant_widget);
        // Update the image
        views.setImageViewResource(R.id.widget_plant_image, imgRes);
        // Widgets allow click handlers to only launch pending intents
        views.setOnClickPendingIntent(R.id.widget_plant_image, pendingIntent);
        // Add the wateringservice click handler
        Intent wateringIntent = new Intent(context, PlantWateringService.class);
        wateringIntent.setAction(PlantWateringService.ACTION_WATER_PLANTS);
        PendingIntent wateringPendingIntent = PendingIntent.getService(context, 0, wateringIntent, PendingIntent.FLAG_UPDATE_CURRENT);
        views.setOnClickPendingIntent(R.id.widget_water_button, wateringPendingIntent);
        // Instruct the widget manager to update the widget
        appWidgetManager.updateAppWidget(appWidgetId, views);
    }
```


#### 2. Move the `updateAppWidget` loop to a new method called `updatePlantWidgets` and pass through the image resource. [[code][2]]
```java
        PlantWateringService.startActionUpdatePlantWidgets(context);
```


#### 4. Call `startActionUpdatePlantWidgets` in `onUpdate` as well as in `AddPlantActivity` and `PlantDetailActivity` (add and delete plants)[[code][4]]
```java
    public static void  updatePlantWidgets(Context context, AppWidgetManager appWidgetManager,
                                           int imgRes, int\[\] appWidgetIds) {
        for (int appWidgetId : appWidgetIds) {
            updateAppWidget(context, appWidgetManager, imgRes, appWidgetId);
        }
    }
```



## _PlantWateringService.java_
#### 3. Create a new action `ACTION_UPDATE_PLANT_WIDGETS` to handle updating widget UI and implement `handleActionUpdatePlantWidgets` to query the plant closest to dying and call `updatePlantWidgets` to refresh widgets. [[code][3]]
```java
    public void handleActionUpdatePlantWidgets() {
        // Query to get the plant that's most in need for water (last watered)
        Uri PLANT_URI = BASE_CONTENT_URI
                .buildUpon()
                .appendPath(PATH_PLANTS)
                .build();
        Cursor cursor = getContentResolver().query(
                PLANT_URI,
                null,
                null,
                null,
                PlantContract.PlantEntry.COLUMN_LAST_WATERED_TIME);


        //Extract the plant details
        int imgRes = R.drawable.grass; //Default image in case our garden is empty
        if (cursor != null && cursor.getCount() > 0) {
            cursor.moveToFirst();
            int createTimeIndex = cursor.getColumnIndex(PlantContract.PlantEntry.COLUMN_CREATION_TIME);
            int waterTimeIndex = cursor.getColumnIndex(PlantContract.PlantEntry.COLUMN_LAST_WATERED_TIME);
            int plantTypeIndex = cursor.getColumnIndex(PlantContract.PlantEntry.COLUMN_PLANT_TYPE);
            long timeNow = System.currentTimeMillis();
            long wateredAt = cursor.getLong(waterTimeIndex);
            long createdAt = cursor.getLong(createTimeIndex);
            int plantType = cursor.getInt(plantTypeIndex);
            cursor.close();
            imgRes = PlantUtils.getPlantImageRes(this, timeNow - createdAt, timeNow - wateredAt, plantType);
        }
        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(this);
        int[] appWidgetIds = appWidgetManager.getAppWidgetIds(
                new ComponentName(this, PlantWidgetProvider.class));


        // Now update all widgets
        PlantWidgetProvider.updatePlantWidgets(this, appWidgetManager, imgRes, appWidgetIds);
    }
```



## _Class Reference_
### [AppWidgetManager](https://developer.android.com/reference/android/appwidget/AppWidgetManager.html)
Updates AppWidget state; gets information about installed AppWidget providers and other AppWidget related state.

|Return Type | Method Name|
|:---------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `int[]` | [getAppWidgetIds](https://developer.android.com/reference/android/appwidget/AppWidgetManager.html#getAppWidgetIds(android.content.ComponentName))`(`[ComponentName](https://developer.android.com/reference/android/content/ComponentName.html) `provider)` <br/>Get the list of appWidgetIds that have been bound to the given AppWidget provider. |


### [ComponentName](https://developer.android.com/reference/android/content/ComponentName.html)
Identifier for a specific application component ([Activity](https://developer.android.com/reference/android/app/Activity.html), [Service](https://developer.android.com/reference/android/app/Service.html), [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html), or [ContentProvider](https://developer.android.com/reference/android/content/ContentProvider.html)) that is available. Two pieces of information, encapsulated here, are required to identify a component: the package (a String) it exists in, and the class (a String) name inside of that package.

|Constructor   |
|:---|
|[ComponentName](https://developer.android.com/reference/android/content/ComponentName.html#ComponentName(android.content.Context,java.lang.String)) `(`[Context](https://developer.android.com/reference/android/content/Context.html) `pkg,` [String](https://developer.android.com/reference/java/lang/String.html) `cls)`<br/>Create a new component identifier from a Context and class name.   |



## _Screenshots_
![1](screenshots/1.gif)





[1]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/30dea4c83bac41f2007dc85b1b22d51305dbd8d6/app/src/main/java/com/example/android/mygarden/PlantWidgetProvider.java#L31-L50
[2]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/30dea4c83bac41f2007dc85b1b22d51305dbd8d6/app/src/main/java/com/example/android/mygarden/PlantWidgetProvider.java#L56
[3]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/30dea4c83bac41f2007dc85b1b22d51305dbd8d6/app/src/main/java/com/example/android/mygarden/PlantWateringService.java#L111-L144
[4]: https://github.com/aaroncrutchfield/AdvancedAndroid_MyGarden/blob/30dea4c83bac41f2007dc85b1b22d51305dbd8d6/app/src/main/java/com/example/android/mygarden/PlantWidgetProvider.java#L61-L66
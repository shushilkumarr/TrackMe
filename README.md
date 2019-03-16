# TrackMe
A simple Application that tracks the location of the user using Fused Location API.

## Configuration

1. Add the following in manifest.xml
```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
2. Include the following in the app gradle

```gradle
implementation 'com.google.android.gms:play-services-location:16.0.0'
```
## Code

1. Check whether GPS is enabled

```java
mLocationManager = (LocationManager) getSystemService(LOCATION_SERVICE);
        if (!mLocationManager.isProviderEnabled(LocationManager.GPS_PROVIDER)) {
            turnGPSOn();
        }
```
2. If not, prompt user to enable and start settings intent

```java
private void turnGPSOn() {

        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("GPS Disabled");
        builder.setMessage("Gps is disabled, in order to use the application properly you need to enable GPS of your device");
        builder.setPositiveButton("Enable GPS", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                startActivityForResult(new Intent(android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS), GPS_ENABLE_REQUEST);
            }
        }).setNegativeButton("No, Just Exit", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {

                finish();

            }
        });
        builder.create().show();

    }
```
3. when GPS enabled, initialize the fused location client, location request and location callback

```java
fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
requestingLocationUpdates = true;
createLocationRequest();
createLocationCallback();
```

```java
private void createLocationRequest() {
        locationRequest = new LocationRequest();
        locationRequest.setInterval(UPDATE_INTERVAL_IN_MILLISECONDS);
        locationRequest.setFastestInterval(FASTEST_UPDATE_INTERVAL_IN_MILLISECONDS);
        locationRequest.setPriority(LocationRequest.PRIORITY_BALANCED_POWER_ACCURACY);
    }
    
private void createLocationCallback() {
        locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(LocationResult locationResult) {
                if (locationResult == null) {
                    return;
                }
                Location location = locationResult.getLastLocation();
                lat = location.getLatitude();
                lng = location.getLongitude();
                current = new LatLng(lat, lng);
                if (mMap != null) {
                    if (marker == null) {
                        marker = mMap.addMarker(new MarkerOptions().position(current).title("Your Current Position"));
                        mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(current, 5.0f));
                    } else
                        marker.setPosition(current);
                    zoom = mMap.getCameraPosition().zoom;
                    mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(current, zoom));
                }
            }

        };
    }
```
4. check whether location permission is gramted by user and start location updates
```java
if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(MapsActivity.this, new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, REQUEST_CODE);
            return;
        }
fusedLocationClient.requestLocationUpdates(locationRequest,
                locationCallback,
                null);
```

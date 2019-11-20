
# Android Geolocation API and Google Maps #

### Part 1:  Create a basic Android Map Application ###

Follow the tutorial to create a simple Android project with Google Maps:

https://developers.google.com/maps/documentation/android-api/start


### Part 2: Accessing the Map object and showing User's location ###

1) Read the documentation about how to get user's last location:
    https://developer.android.com/training/location/retrieve-current
    
2) Make your activity implement the OnMapReadyCallback interface. This interface will allow you to know when the map is ready to use when the onMapReady method is called.

3) Retrieve SupportMapFragment and retrieve the map asynchronously sending the activity as parameter to be the callback:

    ```` java
    // Obtain the SupportMapFragment and get notified when the map is ready to be used.
    SupportMapFragment mapFragment = 
    (SupportMapFragment) getSupportFragmentManager().findFragmentById( R.id.map );
    mapFragment.getMapAsync( this );
    ````

4) Create another field for the GoogleMap object on the Activity and save it when the onMapReady method is called:

    ```` java
    @Override
    public void onMapReady( GoogleMap googleMap )
    {
        this.googleMap = googleMap;
    }
    ```` 

5) Create a method that shows your current location and another to check for the user permission to access the location with the following code:

    ```java
        public void showMyLocation()
        {
           if ( googleMap != null )
           {
               String[] permissions = { android.Manifest.permission.ACCESS_FINE_LOCATION,
                   android.Manifest.permission.ACCESS_COARSE_LOCATION };
               if ( hasPermissions( this, permissions ) )
               {
                   googleMap.setMyLocationEnabled( true );
        
                   FusedLocationProviderClient fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
                   fusedLocationClient.getLastLocation()
                    .addOnSuccessListener(this, new OnSuccessListener<Location>() {
                        @Override
                        public void onSuccess(Location location) {
                            // Got last known location. In some rare situations this can be null.
                            if (location != null) {
                                addMarkerAndZoom( location, "My Location", 15 );
                            }
                        }
                    });
               }
               else
               {
        ActivityCompat.requestPermissions( activity, permissions, ACCESS_LOCATION_PERMISSION_CODE );
               }
           }
        }
        
        public static boolean hasPermissions( Context context, String[] permissions )
        {
           for ( String permission : permissions )
           {
               if ( ContextCompat.checkSelfPermission( context, permission ) == PackageManager.PERMISSION_DENIED )
               {
                   return false;
               }
           }
           return true;
        }
    ``` 


6) Create the method addMarkerAndZoom in the main Activity:

    ```` Java
    public void addMarkerAndZoom( Location location, String title, int zoom  )
    {
       LatLng myLocation = new LatLng( location.getLatitude(), location.getLongitude() );
       googleMap.addMarker( new MarkerOptions().position( myLocation ).title( title ) );
       googleMap.moveCamera( CameraUpdateFactory.newLatLngZoom( myLocation, zoom ) );
    }
    ```` 


7) Override the *onRequestPermissionsResult* method and check for the 
ACCESS_LOCATION_PERMISSION_CODE with the following code:

```` Java

    private final int ACCESS_LOCATION_PERMISSION_CODE = 44;

    @Override
    public void onRequestPermissionsResult( int requestCode, @NonNull String[] permissions,
                                           @NonNull int[] grantResults )
    {
       for ( int grantResult : grantResults )
       {
           if ( grantResult == -1 )
           {
               return;
           }
       }
       switch ( requestCode )
       {
           case ACCESS_LOCATION_PERMISSION_CODE:
               showMyLocation();
               break;
           default:
               super.onRequestPermissionsResult( requestCode, permissions, grantResults );
       }
    }
````

8) Test the application on a real device and make sure your location is shown.

9) Sometimes lastLocation returns null. Therefore you can use the *FusedLocationClient* to actively retrieve the user location. Take a look at the documentation:

https://developer.android.com/training/location/receive-location-updates

### Part 3: Using the GoogleApiClient and Geolocation API ###

Before you get started read and understand the documentation:

https://developer.android.com/training/location/display-address

1) Add a button at the bottom of the map with the label “find address”

![Alt text](https://github.com/COSW-ECI/android-geolocation-api/blob/master/images/map.png)

2) Add an EditText with the id address to the top of the Map view where the address will be displayed.

3) Include the Google Play Services Library on your build.gradle file:
       
    ````Gradle
        implementation 'com.google.android.gms:play-services-location:17.0.0'
    ````

4) Create a method that handles the *onClick* event from the Find Address button with the following code:

    ````Java
        public void onFindAddressClicked( View view )
        {
        startFetchAddressIntentService();
        }
        public void startFetchAddressIntentService()
        {
          fusedLocationClient.getLastLocation()
            .addOnSuccessListener(this, new OnSuccessListener<Location>() {
                @Override
                public void onSuccess(Location location) {
                    if ( location != null )
                    {
                        AddressResultReceiver addressResultReceiver = new AddressResultReceiver( new Handler() );
                        addressResultReceiver.setAddressResultListener( new AddressResultListener()
                        {
                            @Override
                            public void onAddressFound( final String address )
                            {
                                runOnUiThread( new Runnable()
                                {
                                    @Override
                                    public void run()
                                    {
                                        MapsActivity.this.address.setText( address );
                                        MapsActivity.this.address.setVisibility( View.VISIBLE );
                                    }
                                } );


                            }
                        } );
                        Intent intent = new Intent( MapsActivity.this, FetchAddressIntentService.class );
                        intent.putExtra( FetchAddressIntentService.RECEIVER, addressResultReceiver );
                        intent.putExtra( FetchAddressIntentService.LOCATION_DATA_EXTRA, location );
                        startService( intent );
                    }
                }
            });
        }
   ````
   
5) To be able to run the project you will need the following classes *AddressResultReceiver*,  *AddressResultListener* and *FetchAddressIntentService*. In the following url you will be able to find a project that contains the missing classes:

    https://github.com/sancarbar/android-maps-demo
    
6) Include the service you created in the *AndroidManifest* file (below the </activity> closing tag):
    ````xml
        <service
         android:name=".FetchAddressIntentService"
         android:exported="false" />

7) Verify that it works and the address is displayed correctly.

### Part 4: Implement Add location feature ###         

1) Add a floating action button to the Maps view at the right bottom.

2) Implement the *onClick* listener for the Add Button that redirects to another Activity where the user can add a new Location to the map. A location has a name, a description and a geolocation (longitude and latitude).
Tip: You should use *startActivityForResult()* https://developer.android.com/training/basics/intents/result

3) Create a form that captures the Location data and add a save button that validates the form and submits the data.

4) Once the user creates a new Location then the Application should take you back to the map and displayed the created location.




# Android Geolocation API and Google Maps #

### Part 1:  Create a basic Android Map Application ###

Follow the tutorial to create a simple Android project with Google Maps:

https://developers.google.com/maps/documentation/android-api/start


### Part 2: Accessing the Map object and showing User's location ###

1) Make your activity implement the OnMapReadyCallback interface. This interface will allow you to know when the map is ready to use when the onMapReady method is called.

2) Retrieve SupportMapFragment and retrieve the map asynchronously sending the activity as parameter to be the callback:

    ```` java
    // Obtain the SupportMapFragment and get notified when the map is ready to be used.
    SupportMapFragment mapFragment = 
    (SupportMapFragment) getSupportFragmentManager().findFragmentById( R.id.map );
    mapFragment.getMapAsync( this );
    ````

3) Create another field for the GoogleMap object on the Activity and save it when the onMapReady method is called:

    ```` java
    @Override
    public void onMapReady( GoogleMap googleMap )
    {
        this.googleMap = googleMap;
    }
    ```` 

4) Create a method that shows your current location and another to check for the user permission to access the location with the following code:

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
        
                   Location lastLocation = LocationServices.FusedLocationApi.getLastLocation( googleApiClient );
                   if ( lastLocation != null )
                   {
                       addMarkerAndZoom( lastLocation, "My Location", 15 );
                   }
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


5) Create the method addMarkerAndZoom in the main Activity:

    ```` Java
    public void addMarkerAndZoom( Location location, String title, int zoom  )
    {
       LatLng myLocation = new LatLng( location.getLatitude(), location.getLongitude() );
       googleMap.addMarker( new MarkerOptions().position( myLocation ).title( title ) );
       googleMap.moveCamera( CameraUpdateFactory.newLatLngZoom( myLocation, zoom ) );
    }
    ```` 


6) Override the *onRequestPermissionsResult* method and check for the 
ACCESS_LOCATION_PERMISSION_CODE with the following code:

```` Java
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

7) Test the application on a real device and make sure your location is shown.

### Part 3: Using the GoogleApiClient and Geolocation API ###

1) Add a button at the bottom of the map with the label “find address”

![Alt text](https://github.com/COSW-ECI/android-geolocation-api/blob/master/images/map.png)

2) Include the Google Play Services Library on your build.gradle file:
       
    ````Gradle
        compile 'com.google.android.gms:play-services:10.2.1'
    ````

3) Declare a GoogleApiClient field on the main activity and instantiate it on the onCreate method:

    ````Java
        //Configure Google Maps API Objects
        googleApiClient = new GoogleApiClient.Builder( this ).addConnectionCallbacks( this ).
        addOnConnectionFailedListener( this ).addApi( LocationServices.API ).build();
        locationRequest.setInterval( 10000 );
        locationRequest.setFastestInterval( 5000 );
        locationRequest.setPriority( LocationRequest.PRIORITY_BALANCED_POWER_ACCURACY );
        googleApiClient.connect();
    ````

4) Make the main Activity Implement the *GoogleApiClient.ConnectionCallbacks* with the following code:

    ````Java
        @Override
        public void onConnected( @Nullable Bundle bundle )
        {
          LocationServices.FusedLocationApi.requestLocationUpdates( googleApiClient, locationRequest,
                                                                     new LocationListener()
                                                                     {
                                                                         @Override
                                                                         public void onLocationChanged( Location location )
                                                                         {
                                                                             showMyLocation();
                                                                             stopLocationUpdates();
                                                                         }
                                                                     } );
        
        }
        
        @Override
        public void onConnectionSuspended( int i )
        {
           LocationServices.FusedLocationApi.removeLocationUpdates( googleApiClient, null);
        }
        
        public void stopLocationUpdates()
        {
           LocationServices.FusedLocationApi.removeLocationUpdates( googleApiClient, new LocationListener()
           {
               @Override
               public void onLocationChanged( Location location )
               {
        
               }
           } );
        }
    ````

5) Create a method that handles the *onClick* event from the Find Address button with the following code:

    ````Java
        public void onFindAddressClicked( View view )
        {
        startFetchAddressIntentService();
        }
        public void startFetchAddressIntentService()
        {
           Location lastLocation = LocationServices.FusedLocationApi.getLastLocation( googleApiClient );
           if ( lastLocation != null )
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
               Intent intent = new Intent( this, FetchAddressIntentService.class );
               intent.putExtra( FetchAddressIntentService.RECEIVER, addressResultReceiver );
               intent.putExtra( FetchAddressIntentService.LOCATION_DATA_EXTRA, lastLocation );
               startService( intent );
           }
        }
        }
   ````
   
6) To be able to run the project you will need the following classes *AddressResultReceiver*,  *AddressResultListener* and *FetchAddressIntentService*. In the following url you will be able to find a project that contains the missing classes:

    https://github.com/sancarbar/android-maps-demo
    
7) Include the service you created in the *AndroidManifest* file (below the </activity> closing tag):
    ````xml
        <service
         android:name=".FetchAddressIntentService"
         android:exported="false" />
         
### Part 4: Implement Add location feature ###         

1) Add a floating action button to the Maps view at the right bottom.

2) Implement the *onClick* listener for the Add Button that redirects to another Activity where the user can add a new Location to the map. A location has a name, a description and a geolocation (longitude and latitude).

3) Create a form that captures the Location data and add a save button that validates the form and submits the data.

3) Once the user creates a new Location then the Application should take you back to the map and displayed the created location.



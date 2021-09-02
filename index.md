## Welcome to my 1st android app poject!

Hello! I'm Manraj, I had a vision before making this app that all the things which you could possibly need during a COVID emergency should be at one place. And this is how my idea for the app came. An app where you could track COVID cases, Guides on what to do/not to do, Vaccination help, Map for easy guiding to Hospitals, Banks and ATMs, Get info of beds, oxygen, medicine, and more at a single place.
The vision was only partially completed and I will keep on updating the app as my knowldedge in programming increases.

The whole code is on my GitHub. The code below is the MainActivity class.
```markdown
public class MainActivity extends AppCompatActivity implements OnMapReadyCallback, ConnectionCallbacks, OnConnectionFailedListener {
    private Button buttonneg, buttonpos, vaccine;
    SupportMapFragment supportMapFragment;
    private FusedLocationProviderClient mLocationClient;
    private TextView totalConfirm, totalActive, totalRecovered, totalDeath;
    private TextView todayConfirm, todayRecovered, todayDeath, date;
    private PieChart pieChart;
    private List<CountryData> list;
    String country = "India";
    boolean isPermissionGranted;
    GoogleMap map;
    Spinner spType;
    Button btFind;
    FusedLocationProviderClient fusedLocationProviderClient;
    double currentLat = 0, currentLong = 0;


    @SuppressLint("VisibleForTests")
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        list = new ArrayList<>();
        if (getIntent().getStringExtra("country") != null)
            country = getIntent().getStringExtra("country");

        vaccine = findViewById(R.id.vaccination);

        vaccine.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                gotoUrl("https://www.cowin.gov.in/home");
            }
        });

        buttonneg = findViewById(R.id.covidnegative);
        buttonneg.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                openPrecautionIfNegative();
            }
        });

        buttonpos = findViewById(R.id.covidpositive);
        buttonpos.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                openPrecautionIfPositive();
            }
        });

        init();

        TextView cname = findViewById(R.id.cname);
        cname.setText((country));
        cname.setOnClickListener(v ->
                startActivity(new Intent(MainActivity.this, CountryActivity.class)));
        ApiUtilities.getApiInterface().getCountryData().enqueue(new Callback<List<CountryData>>() {
            public void onResponse(Call<List<CountryData>> call, Response<List<CountryData>> response) {
                list.addAll((response.body()));
                for (int i = 0; i < list.size(); i++) {
                    if (list.get(i).getCountry().equals(country)) {
                        int confirm = Integer.parseInt(list.get(i).getCases());
                        int active = Integer.parseInt(list.get(i).getActive());
                        int death = Integer.parseInt(list.get(i).getDeaths());
                        int recovered = Integer.parseInt(list.get(i).getRecovered());

                        totalActive.setText(NumberFormat.getInstance().format(active));
                        totalConfirm.setText(NumberFormat.getInstance().format(confirm));
                        totalRecovered.setText(NumberFormat.getInstance().format(recovered));
                        totalDeath.setText(NumberFormat.getInstance().format(death));
                        todayDeath.setText("+" + NumberFormat.getInstance().format(Integer.parseInt(list.get(i).getTodayDeaths())));
                        todayConfirm.setText("+" + NumberFormat.getInstance().format(Integer.parseInt(list.get(i).getTodayCases())));
                        todayRecovered.setText("+" + NumberFormat.getInstance().format(Integer.parseInt(list.get(i).getTodayRecovered())));

                        setText(list.get(i).getUpdated());
                        pieChart.addPieSlice(new PieModel("Confirm", confirm, getResources().getColor(R.color.yellow)));
                        pieChart.addPieSlice(new PieModel("Active", active, getResources().getColor(R.color.blue)));
                        pieChart.addPieSlice(new PieModel("Death", death, getResources().getColor(R.color.red)));
                        pieChart.addPieSlice(new PieModel("Recovered", confirm, getResources().getColor(R.color.green)));
                        pieChart.startAnimation();
                    }
                }
            }

            public void onFailure(Call<List<CountryData>> call, Throwable t) {
                Toast.makeText(MainActivity.this, "Error" + t.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });


        checkMyPermission();

        initMap();
        mLocationClient = new FusedLocationProviderClient(this);
        getCurrLoc();


        spType = findViewById(R.id.sp_type);
        btFind = findViewById(R.id.bt_find);
        supportMapFragment = (SupportMapFragment) getSupportFragmentManager().findFragmentById(R.id.google_map);
        String[] placeTypeList = {"hospital", "bank", "atm"};
        String[] placeNameList = {"Hospital", "Bank", "ATM"};
        spType.setAdapter(new ArrayAdapter<>(MainActivity.this,
                android.R.layout.simple_spinner_dropdown_item, placeNameList));

        fusedLocationProviderClient = LocationServices.getFusedLocationProviderClient(this);
        if (ActivityCompat.checkSelfPermission(MainActivity.this
                , Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            getCurrentLocation();
        } else {
            ActivityCompat.requestPermissions(MainActivity.this
                    , new String[]{Manifest.permission.ACCESS_FINE_LOCATION}, 44);
        }
        btFind.setOnClickListener(new View.OnClickListener() {
            public void onClick(View v) {
                int i = spType.getSelectedItemPosition();
                String url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json" +
                        "?location" + currentLat + "," + currentLong +
                        "&radius=1005000" +
                        "&types=" + placeTypeList[i] +
                        "&sensor=true" +
                        "&key" + getResources().getString(R.string.google_maps_key);
                new PlaceTask().execute(url);
            }
        });


    }

    private void getCurrentLocation() {
        @SuppressLint("MissingPermission")
        Task<Location> task = fusedLocationProviderClient.getLastLocation();
        task.addOnSuccessListener(new OnSuccessListener<Location>() {
            @Override
            public void onSuccess(Location location) {
                if (location != null) {
                    currentLat = location.getLatitude();
                    currentLong = location.getLongitude();
                    supportMapFragment.getMapAsync(new OnMapReadyCallback() {
                        @Override
                        public void onMapReady(@NonNull @NotNull GoogleMap googleMap) {
                            map.animateCamera(CameraUpdateFactory.newLatLngZoom(
                                    new LatLng(currentLat, currentLong), 20
                            ));
                        }
                    });
                }
            }
        });
    }


    private void initMap() {
        if (isPermissionGranted) {
            SupportMapFragment supportMapFragment = (SupportMapFragment) getSupportFragmentManager().findFragmentById(R.id.google_map);
            supportMapFragment.getMapAsync(this);
        }
    }

    @SuppressLint("MissingPermission")
    private void getCurrLoc() {
        mLocationClient.getLastLocation().addOnCompleteListener(task -> {
            if (task.isSuccessful()) {
                Location location = task.getResult();
                gotoLocation(location.getLatitude(), location.getLongitude());
            }
        });
    }

    private void gotoLocation(double latitude, double longitude) {
        LatLng latlng = new LatLng(latitude, longitude);
        CameraUpdate cameraUpdate = CameraUpdateFactory.newLatLngZoom(latlng, 20);
        map.moveCamera(cameraUpdate);
        map.setMapType(GoogleMap.MAP_TYPE_NORMAL);
    }

    private void checkMyPermission() {
        Dexter.withContext(this).withPermission(Manifest.permission.ACCESS_FINE_LOCATION).withListener(new PermissionListener() {
            @Override
            public void onPermissionGranted(PermissionGrantedResponse permissionGrantedResponse) {
                Toast.makeText(MainActivity.this, "permission granted", Toast.LENGTH_SHORT).show();
                isPermissionGranted = true;
            }

            @Override
            public void onPermissionDenied(PermissionDeniedResponse permissionDeniedResponse) {
                Intent intent = new Intent();
                intent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                Uri uri = Uri.fromParts("package", getPackageName(), "");
                intent.setData(uri);
                startActivity(intent);
            }

            @Override
            public void onPermissionRationaleShouldBeShown(PermissionRequest permissionRequest, PermissionToken permissionToken) {
                permissionToken.continuePermissionRequest();
            }
        }).check();
    }


    private void gotoUrl(String s) {
        Uri uri = Uri.parse(s);
        startActivity(new Intent(Intent.ACTION_VIEW, uri));
    }

    public void openPrecautionIfNegative() {
        Intent intent = new Intent(this, PrecautionIfNegative.class);
        startActivity(intent);
    }

    public void openPrecautionIfPositive() {
        Intent intent = new Intent(this, PrecautionIfPositive.class);
        startActivity(intent);
    }

    private void setText(String updated) {
        DateFormat format = new SimpleDateFormat("dd/MM/yy");
        long milliseconds = Long.parseLong(updated);
        Calendar calendar = Calendar.getInstance();
        calendar.setTimeInMillis(milliseconds);
        date.setText("Updated at " + format.format(calendar.getTime()));
    }

    private void init() {
        totalConfirm = findViewById(R.id.totalConfirm);
        totalActive = findViewById(R.id.totalActive);
        totalRecovered = findViewById(R.id.totalRecovered);
        totalDeath = findViewById(R.id.totalDeath);
        todayConfirm = findViewById(R.id.todayConfirm);
        todayRecovered = findViewById(R.id.todayRecovered);
        todayDeath = findViewById(R.id.todayDeath);
        pieChart = findViewById(R.id.pieChart);
        date = findViewById(R.id.date);

    }

    @SuppressLint("MissingPermission")
    @Override
    public void onMapReady(GoogleMap googleMap) {
        map = googleMap;
        map.setMyLocationEnabled(true);
        map.setMapType(GoogleMap.MAP_TYPE_NORMAL);
        map.getUiSettings().setZoomControlsEnabled(true);
        map.getUiSettings().setZoomGesturesEnabled(true);
        map.getUiSettings().setCompassEnabled(false);
        map.getUiSettings().setScrollGesturesEnabled(true);
        map.getUiSettings().setScrollGesturesEnabledDuringRotateOrZoom(false);
        map.setMyLocationEnabled(true);
        map.animateCamera(CameraUpdateFactory.zoomTo(8f));
    }


    @Override
    public void onConnected(@Nullable Bundle bundle) {

    }

    @Override
    public void onConnectionSuspended(int i) {

    }

    @Override
    public void onConnectionFailed(@NonNull @NotNull ConnectionResult connectionResult) {

    }


    @SuppressLint("MissingSuperCall")
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {

        if (requestCode == 44) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                getCurrentLocation();
            }
        }
    }


    private class PlaceTask extends AsyncTask<String, Integer, String> {
        @Override
        protected String doInBackground(String... strings) {
            String data=null;
            try {
                data=downloadUrl(strings[0]);
            } catch (IOException e) {
                e.printStackTrace();
            }
            return data;
        }

        @Override
        protected void onPostExecute(String s) {
            super.onPostExecute(s);
            new ParserTask().execute(s);
        }
    }

    private String downloadUrl(String string)throws IOException {
        URL url=new URL(string);
        HttpURLConnection connection=(HttpURLConnection)url.openConnection();
        connection.connect();
        InputStream stream=connection.getInputStream();
        BufferedReader reader=new BufferedReader(new InputStreamReader(stream));
        StringBuilder builder=new StringBuilder();
        String line="";
        while ((line =reader.readLine())!=null){
            builder.append(line);

        }
        String data=builder.toString();
        reader.close();
        return data;
    }

    private class ParserTask extends AsyncTask<String,Integer,List<HashMap<String,String>>>{
        @Override
        protected List<HashMap<String, String>> doInBackground(String... strings) {
            JsonParser jsonParser=new JsonParser();
            List<HashMap<String,String>>mapList=null;
            JSONObject object=null;
            try {
                object=new JSONObject(strings[0]);
                mapList=jsonParser.parseResult(object);
            } catch (JSONException e) {
                e.printStackTrace();
            }
            return mapList;
        }

        @Override
        protected void onPostExecute(List<HashMap<String, String>> hashMaps) {
            map.clear();
            for(int i=0;i<hashMaps.size();i++){
                HashMap<String,String> hashMapList=hashMaps.get(i);
                double lat=Double.parseDouble(hashMapList.get("lat"));
                double lng= Double.parseDouble(hashMapList.get("lng"));
                String name=hashMapList.get("name");
                LatLng latLng=new LatLng(lat,lng);
                MarkerOptions options=new MarkerOptions();
                options.position(latLng);
                options.title(name);
                map.addMarker(options);
            }
        }
    }
}
```
##Thank You!


import android.content.DialogInterface;
import android.content.Intent;
import android.graphics.Typeface;
import android.net.Uri;
import android.os.Bundle;
import android.text.Html;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ArrayAdapter;
import android.widget.BaseAdapter;
import android.widget.ListView;
import androidx.appcompat.widget.SearchView;
import android.widget.TextView;
import android.widget.Toast;

import com.google.android.gms.ads.AdRequest;
import com.google.android.gms.ads.AdSize;
import com.google.android.gms.ads.AdView;
import com.google.android.gms.ads.MobileAds;
import com.google.android.gms.ads.initialization.InitializationStatus;
import com.google.android.gms.ads.initialization.OnInitializationCompleteListener;
import com.google.android.material.navigation.NavigationView;
import com.startapp.sdk.adsbase.StartAppAd;
import com.startapp.sdk.adsbase.StartAppSDK;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.ArrayList;

import androidx.appcompat.app.ActionBarDrawerToggle;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.Toolbar;
import androidx.core.view.GravityCompat;
import androidx.drawerlayout.widget.DrawerLayout;


public class MainActivity extends AppCompatActivity implements NavigationView.OnNavigationItemSelectedListener {
    private AdView mAdView;

    ListView listView;
    ArrayList<List_itme> list_itmes = new ArrayList<>();
    ArrayAdapter<List_itme> arrayAdapter;


    String list_type = "main_index";
    
    String Book_id;
    private Toolbar toolbar;
    private DrawerLayout drawerLayout;
    private NavigationView navigationView;
    private static long time;

    TextView textView_Title,textViewto;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);






        toolbar = (Toolbar) findViewById(R.id.main_toolbar);
        setSupportActionBar(toolbar);

        drawerLayout = findViewById(R.id.drawer_layout);
        navigationView = findViewById(R.id.maine_nav);

        ActionBarDrawerToggle actionBarDrawerToggle = new ActionBarDrawerToggle(
                this,
                drawerLayout,
                toolbar,
                R.string.openNavDrawer,
                R.string.closeNavDrawer

        );

        drawerLayout.addDrawerListener(actionBarDrawerToggle);
        actionBarDrawerToggle.syncState();
        navigationView.setNavigationItemSelectedListener(this);










        listView = findViewById(R.id.listView);
        Index("index.txt");//ملف الفهرس الرئيسي

    }

    @Override//هذا اذا ضغط السهم لايخرج من التطبيق
    public void onBackPressed() {//هذا اذا ضغط السهم لايخرج من التطبيق
        if (list_type.equals("sub_index")){//اذا كان في القائمة الفرعية
            Index("index.txt");//يعدنا اللى الرئيسية
            list_type="main_index";//قبل اضافته قمت بالرجوع لكن لم اتمكن من الضغط مرة اخرى ةلم يخرج من التطبيق بواسطة الزر

            onResume();
        } else {
            DrawerLayout drawer = findViewById(R.id.drawer_layout);
            if (drawer.isDrawerOpen(GravityCompat.START)) {
                drawer.closeDrawer(GravityCompat.START);
            } else {
                if (time + 2000 > System.currentTimeMillis()) {
                    super.onBackPressed();
                } else {
                    Toast.makeText(MainActivity.this, R.string.Click_again_to_close, Toast.LENGTH_LONG).show();
                }

                StartAppAd.onBackPressed(this); //اعلان والاسفل
                super.onBackPressed();

                time = System.currentTimeMillis();
            }
        }
    }


    public void Index(String index_type) {
        list_itmes.clear();//كود تنظيف القائمة السابقة
        try {
            InputStream inputStream = getAssets().open(index_type);
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

            String line ;
            int id=0;
            while ((line=bufferedReader.readLine())!=null){
                id++;
                list_itmes.add(new List_itme(line,"book_"+id)); //اسم الفولدر للمجلد الفرعي

            }


        } catch (IOException e) {
            e.printStackTrace();
        }

        ListAdapter adapter = new ListAdapter(list_itmes);
        listView.setAdapter(adapter);

    }
    @SuppressWarnings("StatementWithEmptyBody")
    @Override
    public boolean onNavigationItemSelected(MenuItem item) {          //من هنا يبدأ أمر تفعيل أورار القائمة الجانبية

   

        }

        DrawerLayout drawer = findViewById(R.id.drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
    }

    @Override
    public void onPointerCaptureChanged(boolean hasCapture) {

    }




    class ListAdapter extends BaseAdapter {
        ArrayList<List_itme>list = new ArrayList<>();

        public ListAdapter(ArrayList<List_itme> list) {
            this.list = list;
        }

        @Override
        public int getCount() {
            return list.size();
        }

        @Override
        public Object getItem(int i) {
            return list.get(i);
        }

        @Override
        public long getItemId(int i) {
            return i;
        }

        @Override
        public View getView(final int i, View convertView, ViewGroup parent) {
            View view1 = View.inflate(getApplicationContext(), R.layout.row_itme,null);
            TextView Title = view1.findViewById(R.id.textView);
            Title.setText((CharSequence) list.get(i).getTitle());

            Typeface typefaces = Typeface.createFromAsset(getAssets(), "font1.ttf");//السطر هذا واسفله لتغير خط القائمة الرئيسية والفرعية
            Title.setTypeface(typefaces);


            Title.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {

                    if (list_type.equals("main_index")){ list_type="sub_index";
                        Book_id = list.get(i).getFolder_id();

                        Index(list.get(i).getFolder_id() + "/index.txt");//اسم الفهرس الفرعي

                    }else if(list_type.equals("sub_index")) {//هذا لانتقال الفهرس الفرعي الى صفحة الويب

                        Intent intent = new Intent(MainActivity.this,Web_Activity.class);
                        String line ;//ضفته كي تبدأ الفحات بالرقم1
                        int id=0;//ضفته كي تبدأ الفحات بالرقم1
                        id++;//ضفته كي تبدأ الفحات بالرقم1
                        intent.putExtra("link",Book_id+"/html/"+i+".htm");
                        startActivity(intent);
                    }
                }
            });



            return view1;
        }
    }
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {

            getMenuInflater().inflate(R.menu.main, menu);






            /////////////////////////SearchView////////////////////////////////////
        arrayAdapter = new ArrayAdapter<List_itme>(this,android.R.layout.simple_list_item_1,list_type);
        listView.setAdapter(arrayAdapter);

            MenuItem searchItem = menu.findItem(R.id.menu_searchable);      //هذا لتفعيل ايٌقونة البحث
            SearchView searchView = (SearchView) searchItem.getActionView();
            searchView.setQueryHint("ماذا يجول في ذهنك");

            searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
                @Override
                public boolean onQueryTextSubmit(String query) {
                    return false;
                }

                @Override
                public boolean onQueryTextChange(String newText) {
                    arrayAdapter.getFilter().filter(newText);

                    return false;
                }
            });

            return true;
        }
////////////////////////////////////////////////SearchView////////////////////////////////////////////////////////////////////







    @Override
    public boolean onOptionsItemSelected(MenuItem item) {    
        int id = item.getItemId();


        if (id == R.id.help) {
            try {
                InputStream inputStream = getAssets().open("help.txt");
                InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
                BufferedReader BR = new BufferedReader(inputStreamReader);
                String line;
                StringBuilder msg = new StringBuilder();
                while ((line = BR.readLine()) != null) {
                    msg.append(line + "\n");
                }
                AlertDialog.Builder build = new AlertDialog.Builder(MainActivity.this);
                build.setTitle(R.string.help);
                build.setIcon(R.drawable.icon);
                build.setMessage(Html.fromHtml(msg + ""));
                build.setNegativeButton(R.string.dilog_close, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int id) {
                        //Negative
                    }
                }).show();

            } catch (IOException e) {
                e.printStackTrace();
            }
            return true;
        }
        if (id == R.id.Close) {
            finish();
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

}

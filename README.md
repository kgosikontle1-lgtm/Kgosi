package com.kgosi.marketanalysis;

import android.os.Bundle;
import android.widget.LinearLayout;
import android.widget.TextView;
import android.view.Gravity;
import android.graphics.Color;
import androidx.appcompat.app.AppCompatActivity;

import org.json.JSONObject;
import java.io.InputStreamReader;
import java.io.BufferedReader;
import java.net.HttpURLConnection;
import java.net.URL;

public class MainActivity extends AppCompatActivity {

    TextView signalText;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // --- Create Layout Programmatically ---
        LinearLayout layout = new LinearLayout(this);
        layout.setOrientation(LinearLayout.VERTICAL);
        layout.setGravity(Gravity.CENTER);
        layout.setPadding(50, 50, 50, 50);
        layout.setBackgroundColor(Color.parseColor("#ffffff"));

        signalText = new TextView(this);
        signalText.setText("Fetching forex data...");
        signalText.setTextSize(20f);
        signalText.setGravity(Gravity.CENTER);

        layout.addView(signalText);
        setContentView(layout);

        // --- Fetch Forex Data in Background Thread ---
        new Thread(() -> {
            try {
                // Free API for EUR/USD
                URL url = new URL("https://api.exchangerate.host/latest?base=EUR&symbols=USD");
                HttpURLConnection conn = (HttpURLConnection) url.openConnection();
                conn.setRequestMethod("GET");
                BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                StringBuilder result = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    result.append(line);
                }
                reader.close();

                JSONObject json = new JSONObject(result.toString());
                double rate = json.getJSONObject("rates").getDouble("USD");

                // Simple signal logic
                String signal = rate > 1.05 ? "SELL" : "BUY";

                runOnUiThread(() -> signalText.setText(
                        "Kgosi Market Analysis\n\nEUR/USD Rate: " + rate + "\nSignal: " + signal
                ));

            } catch (Exception e) {
                e.printStackTrace();
                runOnUiThread(() -> signalText.setText("Error fetching data"));
            }
        }).start();
    }
}

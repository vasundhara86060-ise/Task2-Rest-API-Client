import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;
import java.time.ZonedDateTime;
import java.time.ZoneId;
import java.time.format.DateTimeFormatter;

public class Main {
    public static void main(String[] args) {
        try {
            String apiUrl = "https://api.open-meteo.com/v1/forecast?latitude=12.97&longitude=77.59&current_weather=true";

            URL url = new URL(apiUrl);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");

            if (conn.getResponseCode() == 200) {
                BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
                StringBuilder response = new StringBuilder();
                String line;
                while ((line = reader.readLine()) != null) {
                    response.append(line);
                }
                reader.close();

                String json = response.toString();

                // Extract data from the current_weather block
                String currentWeatherPart = json.split("\"current_weather\":\\{")[1].split("\\}")[0];
                String temperature = currentWeatherPart.split("\"temperature\":")[1].split(",")[0];
                String windspeed = currentWeatherPart.split("\"windspeed\":")[1].split(",")[0];

                // ✅ Use current system time in IST instead of API snapshot
                ZonedDateTime nowIST = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));
                DateTimeFormatter outputFormatter = DateTimeFormatter.ofPattern("dd-MM-yyyy hh:mm a");
                String formattedTime = nowIST.format(outputFormatter);

                System.out.println("===== Weather Data =====");
                System.out.println("Time (IST)  : " + formattedTime);
                System.out.println("Temperature : " + temperature + " °C");
                System.out.println("Wind Speed  : " + windspeed + " km/h");
                System.out.println("========================");

            } else {
                System.out.println("HTTP Error: " + conn.getResponseCode());
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

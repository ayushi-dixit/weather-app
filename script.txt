const apiKey = "fcd3449bbb4191a1bef41435f79c2af0"; // OpenWeatherMap API Key

const cityInput = document.getElementById("cityInput");
const searchBtn = document.getElementById("searchBtn");

const city = document.getElementById("city");
const temp = document.getElementById("temp");
const condition = document.getElementById("condition");
const humidity = document.getElementById("humidity");
const wind = document.getElementById("wind");
const icon = document.getElementById("icon");
const dateTime = document.getElementById("dateTime");

const loader = document.getElementById("loader");
const error = document.getElementById("error");

searchBtn.addEventListener("click", getWeather);

async function getWeather() {
    const cityName = cityInput.value.trim();

    if (cityName === "") {
        error.textContent = "Please enter a city name";
        return;
    }

    loader.style.display = "block";
    error.textContent = "";

    try {
        // 🌍 STEP 1: GET LAT & LON (FIX ACCURACY ISSUE)
        const geoResponse = await fetch(
            `https://api.openweathermap.org/geo/1.0/direct?q=${cityName}&limit=1&appid=${apiKey}`
        );

        const geoData = await geoResponse.json();

        if (!geoData.length) {
            throw new Error("City not found");
        }

        const lat = geoData[0].lat;
        const lon = geoData[0].lon;

        // 🌤️ STEP 2: CURRENT WEATHER
        const response = await fetch(
            `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric`
        );

        const data = await response.json();

        city.textContent = data.name;
        temp.innerHTML = `Temperature: ${data.main.temp} °C <br>
<span style="font-size:12px; color:gray;">
Note: Temperature is approximate and may vary slightly.
</span>`;
        condition.textContent = `Condition: ${data.weather[0].description}`;
        humidity.textContent = `Humidity: ${data.main.humidity}%`;
        wind.textContent = `Wind Speed: ${data.wind.speed} m/s`;

        icon.src = `https://openweathermap.org/img/wn/${data.weather[0].icon}@2x.png`;
        icon.alt = data.weather[0].description;
        icon.style.display = "block";

        // 📅 DATE + DAY
        const now = new Date();
        const dayName = now.toLocaleDateString("en-US", { weekday: "long" });
        dateTime.textContent = `${dayName}, ${now.toLocaleString()}`;

        // 🌦️ STEP 3: FORECAST
        const forecastResponse = await fetch(
            `https://api.openweathermap.org/data/2.5/forecast?lat=${lat}&lon=${lon}&appid=${apiKey}&units=metric`
        );

        const forecastData = await forecastResponse.json();

        displayForecast(forecastData);

    } catch (err) {
        error.textContent = err.message;
    } finally {
        loader.style.display = "none";
    }
}

// 📊 5-DAY FORECAST (NEXT 5 DAYS ONLY, SKIP TODAY)
function displayForecast(data) {

    const forecastContainer = document.getElementById("forecast");
    forecastContainer.innerHTML = "";

    const today = new Date().toDateString();
    const daysMap = new Map();

    data.list.forEach(item => {

        const date = new Date(item.dt_txt);
        const day = date.toDateString();

        if (day === today) return;

        if (!daysMap.has(day)) {
            daysMap.set(day, item);
        } else {
            const existing = daysMap.get(day);
            const existingHour = new Date(existing.dt_txt).getHours();
            const currentHour = date.getHours();

            if (Math.abs(currentHour - 12) < Math.abs(existingHour - 12)) {
                daysMap.set(day, item);
            }
        }
    });

    const forecasts = Array.from(daysMap.values()).slice(0, 5);

    forecasts.forEach(day => {

        const date = new Date(day.dt_txt);

        const formattedDate = date.toLocaleDateString("en-US", {
            weekday: "short",
            day: "numeric",
            month: "short"
        });

        forecastContainer.innerHTML += `
            <div class="forecast-card">
                <h4>${formattedDate}</h4>

                <img src="https://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png">

                <p><strong>${Math.round(day.main.temp)}°C</strong></p>

                <p>${day.weather[0].description}</p>

                <p>Humidity: ${day.main.humidity}%</p>
            </div>
        `;
    });
}
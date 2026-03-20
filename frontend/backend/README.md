import { Router, type IRouter } from "express";
import {
  GetWeatherQueryParams, GetRiskQueryParams, GetPayoutQueryParams,
  GetWeatherResponse, GetRiskResponse, GetPayoutResponse,
} from "@workspace/api-zod";

const router: IRouter = Router();

const OPENWEATHER_API_KEY = process.env.OPENWEATHER_API_KEY;
const BASE_URL = "https://api.openweathermap.org/data/2.5/weather";

// ─── Types ───────────────────────────────────────────────────────────────────

interface WeatherApiResponse {
  name: string;
  weather: Array<{ main: string; description: string; icon: string }>;
  main: { temp: number; humidity: number };
  wind: { speed: number };
  cod: number | string;
  message?: string;
}

// ─── Live API fetch (used if OPENWEATHER_API_KEY is set) ─────────────────────

async function fetchWeatherFromApi(city: string): Promise<WeatherApiResponse> {
  if (!OPENWEATHER_API_KEY) throw new Error("NO_API_KEY");
  const url = `${BASE_URL}?q=${encodeURIComponent(city)}&appid=${OPENWEATHER_API_KEY}&units=metric`;
  const response = await fetch(url);
  const data = (await response.json()) as WeatherApiResponse;
  if (data.cod === "404" || data.cod === 404) throw new Error("CITY_NOT_FOUND");
  if (data.cod !== 200 && data.cod !== "200") throw new Error("API_ERROR");
  return data;
}

// ─── Mock data fallback ───────────────────────────────────────────────────────
// Deterministic per city name (hash-based), so same city always returns same result

function getMockWeather(city: string) {
  const conditions = [
    { condition: "Rain",        description: "moderate rain",             icon: "10d", temp: 18,  humidity: 82, windSpeed: 5.5 },
    { condition: "Clouds",      description: "overcast clouds",           icon: "04d", temp: 22,  humidity: 65, windSpeed: 3.2 },
    { condition: "Clear",       description: "clear sky",                 icon: "01d", temp: 30,  humidity: 40, windSpeed: 2.1 },
    { condition: "Thunderstorm",description: "thunderstorm with rain",    icon: "11d", temp: 16,  humidity: 90, windSpeed: 8.3 },
    { condition: "Snow",        description: "light snow",                icon: "13d", temp: -2,  humidity: 78, windSpeed: 4.0 },
    { condition: "Drizzle",     description: "light intensity drizzle",   icon: "09d", temp: 20,  humidity: 75, windSpeed: 2.8 },
  ];

  const hash = city.toLowerCase().split("").reduce((acc, char) => acc + char.charCodeAt(0), 0);
  const mock = conditions[hash % conditions.length];

  return {
    city: city.charAt(0).toUpperCase() + city.slice(1),
    condition: mock.condition,
    description: mock.description,
    temperature: mock.temp,
    humidity: mock.humidity,
    windSpeed: mock.windSpeed,
    icon: mock.icon,
  };
}

// ─── Risk calculation ─────────────────────────────────────────────────────────

function calculateRisk(condition: string) {
  const conditionMap: Record<string, { level: "Low" | "Medium" | "High"; score: number; reason: string }> = {
    Rain:        { level: "High",   score: 85,  reason: "Rainy conditions significantly reduce delivery efficiency and increase accident risk." },
    Drizzle:     { level: "High",   score: 75,  reason: "Drizzle makes roads slippery and reduces visibility for delivery partners." },
    Thunderstorm:{ level: "High",   score: 95,  reason: "Thunderstorm conditions are dangerous for delivery operations with risk of lightning and flooding." },
    Snow:        { level: "High",   score: 90,  reason: "Snow creates hazardous road conditions severely impacting delivery safety and speed." },
    Ash:         { level: "High",   score: 80,  reason: "Ash in the air creates hazardous conditions for both drivers and vehicles." },
    Squall:      { level: "High",   score: 88,  reason: "Squalls create sudden dangerous weather making deliveries unsafe." },
    Tornado:     { level: "High",   score: 100, reason: "Tornado conditions are extremely dangerous. Do not deliver." },
    Clouds:      { level: "Medium", score: 45,  reason: "Cloudy conditions may reduce demand slightly but generally safe for deliveries." },
    Mist:        { level: "Medium", score: 55,  reason: "Misty conditions reduce visibility and slow down delivery times." },
    Fog:         { level: "Medium", score: 60,  reason: "Foggy conditions significantly reduce visibility and require extra caution." },
    Haze:        { level: "Medium", score: 50,  reason: "Hazy conditions may affect visibility but deliveries can continue with care." },
    Smoke:       { level: "Medium", score: 55,  reason: "Smoky conditions can affect air quality and reduce delivery efficiency." },
    Dust:        { level: "Medium", score: 50,  reason: "Dusty conditions may slow deliveries and affect vehicle performance." },
    Sand:        { level: "Medium", score: 52,  reason: "Sandy conditions can make driving difficult and affect delivery times." },
    Clear:       { level: "Low",    score: 15,  reason: "Clear weather is ideal for deliveries with high demand and safe conditions." },
  };

  return conditionMap[condition] || { level: "Medium" as const, score: 50, reason: "Weather conditions may affect delivery operations." };
}

// ─── Payout calculation ───────────────────────────────────────────────────────

function calculatePayout(riskLevel: "Low" | "Medium" | "High", riskScore: number) {
  const dailyIncome = 120; // baseline daily income in USD

  let incomeLossPercent = 0;
  if (riskLevel === "High")        incomeLossPercent = 0.6 + (riskScore - 75) / 100;
  else if (riskLevel === "Medium") incomeLossPercent = 0.25 + (riskScore - 40) / 200;
  else                             incomeLossPercent = 0.05;

  const estimatedIncomeLoss = Math.round(dailyIncome * Math.min(incomeLossPercent, 1) * 100) / 100;
  const suggestedPayout     = Math.round(estimatedIncomeLoss * 1.2 * 100) / 100;

  return { estimatedIncomeLoss, suggestedPayout, autoPayout: riskLevel === "High" };
}

// ─── Shared helper to get weather (live or mock) ──────────────────────────────

async function resolveWeather(city: string, res: any) {
  if (OPENWEATHER_API_KEY) {
    try {
      const apiData = await fetchWeatherFromApi(city);
      return {
        city: apiData.name,
        condition: apiData.weather[0].main,
        description: apiData.weather[0].description,
        temperature: Math.round(apiData.main.temp * 10) / 10,
        humidity: apiData.main.humidity,
        windSpeed: Math.round(apiData.wind.speed * 10) / 10,
        icon: apiData.weather[0].icon,
      };
    } catch (err) {
      if ((err as Error).message === "CITY_NOT_FOUND") {
        res.status(404).json({ error: "NOT_FOUND", message: `City "${city}" not found` });
        return null;
      }
      // Fall through to mock on other API errors
    }
  }
  return getMockWeather(city);
}

// ─── Routes ───────────────────────────────────────────────────────────────────

// GET /api/weather?city=London
router.get("/weather", async (req, res) => {
  const parsed = GetWeatherQueryParams.safeParse(req.query);
  if (!parsed.success) { res.status(400).json({ error: "BAD_REQUEST", message: "city is required" }); return; }

  try {
    const weather = await resolveWeather(parsed.data.city, res);
    if (!weather) return;
    res.json(GetWeatherResponse.parse(weather));
  } catch {
    res.status(500).json({ error: "INTERNAL_ERROR", message: "Failed to fetch weather data" });
  }
});

// GET /api/risk?city=London
router.get("/risk", async (req, res) => {
  const parsed = GetRiskQueryParams.safeParse(req.query);
  if (!parsed.success) { res.status(400).json({ error: "BAD_REQUEST", message: "city is required" }); return; }

  try {
    const weather = await resolveWeather(parsed.data.city, res);
    if (!weather) return;
    const risk = calculateRisk(weather.condition);
    res.json(GetRiskResponse.parse({ city: weather.city, riskLevel: risk.level, riskScore: risk.score, reason: risk.reason, weather }));
  } catch {
    res.status(500).json({ error: "INTERNAL_ERROR", message: "Failed to calculate risk" });
  }
});

// GET /api/payout?city=London  ← main endpoint used by the frontend
router.get("/payout", async (req, res) => {
  const parsed = GetPayoutQueryParams.safeParse(req.query);
  if (!parsed.success) { res.status(400).json({ error: "BAD_REQUEST", message: "city is required" }); return; }

  try {
    const weather = await resolveWeather(parsed.data.city, res);
    if (!weather) return;
    const risk   = calculateRisk(weather.condition);
    const payout = calculatePayout(risk.level, risk.score);
    res.json(GetPayoutResponse.parse({
      city: weather.city,
      riskLevel: risk.level,
      ...payout,
      weather,
      risk: { city: weather.city, riskLevel: risk.level, riskScore: risk.score, reason: risk.reason, weather },
    }));
  } catch {
    res.status(500).json({ error: "INTERNAL_ERROR", message: "Failed to calculate payout" });
  }
});

export default router;

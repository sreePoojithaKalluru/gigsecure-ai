import React, { useState } from "react";
import { useGetPayout } from "@workspace/api-client-react";
import { motion, AnimatePresence } from "framer-motion";
import { 
  ShieldAlert, Cloud, Droplets, Wind, 
  ShieldCheck, TrendingDown, CheckCircle2, 
  MapPin, Loader2, AlertTriangle, CloudRain,
  Sun, CloudLightning, Snowflake, ArrowRight
} from "lucide-react";
import { cn } from "../lib/utils";
import { Card } from "../components/ui/card";

export default function Home() {
  const [city, setCity] = useState("");
  const [submittedCity, setSubmittedCity] = useState("");

  const { data, isLoading, isError } = useGetPayout(
    { city: submittedCity },
    { query: { enabled: !!submittedCity, retry: false } }
  );

  const handleSearch = (e: React.FormEvent) => {
    e.preventDefault();
    if (city.trim()) setSubmittedCity(city.trim());
  };

  const getWeatherIcon = (condition?: string) => {
    if (!condition) return <Cloud className="w-16 h-16 text-slate-300" />;
    const c = condition.toLowerCase();
    if (c.includes("rain") || c.includes("drizzle")) return <CloudRain className="w-16 h-16 text-blue-500" />;
    if (c.includes("cloud")) return <Cloud className="w-16 h-16 text-slate-400" />;
    if (c.includes("clear")) return <Sun className="w-16 h-16 text-amber-400" />;
    if (c.includes("snow")) return <Snowflake className="w-16 h-16 text-sky-300" />;
    if (c.includes("thunder")) return <CloudLightning className="w-16 h-16 text-indigo-500" />;
    return <Cloud className="w-16 h-16 text-slate-400" />;
  };

  const riskTheme = {
    Low:    { badge: "bg-emerald-100 text-emerald-800 border-emerald-200", icon: <CheckCircle2 className="w-6 h-6 text-emerald-600" />, color: "text-emerald-500" },
    Medium: { badge: "bg-amber-100 text-amber-800 border-amber-200",       icon: <AlertTriangle className="w-6 h-6 text-amber-600" />,  color: "text-amber-500"   },
    High:   { badge: "bg-rose-100 text-rose-800 border-rose-200",          icon: <ShieldAlert className="w-6 h-6 text-rose-600" />,     color: "text-rose-500"    },
  };

  return (
    <div className="min-h-screen bg-slate-50 font-sans pb-20">
      {/* Hero */}
      <div className="relative w-full overflow-hidden bg-slate-950 pb-40 pt-20 shadow-2xl">
        <div className="absolute inset-0 opacity-50 mix-blend-screen pointer-events-none">
          <img src={`${import.meta.env.BASE_URL}images/hero-bg.png`} className="w-full h-full object-cover" alt="" />
        </div>
        <div className="absolute inset-0 bg-gradient-to-b from-transparent via-slate-950/50 to-slate-950 pointer-events-none" />

        <div className="relative z-10 max-w-5xl mx-auto px-6 text-center">
          <motion.div initial={{ opacity: 0, y: -20 }} animate={{ opacity: 1, y: 0 }}
            className="inline-flex items-center gap-2 px-4 py-2 rounded-full bg-white/10 backdrop-blur-md border border-white/10 mb-8">
            <ShieldCheck className="w-5 h-5 text-teal-400" />
            <span className="text-white font-medium text-sm">GigSecure AI</span>
          </motion.div>

          <motion.h1 initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.1 }}
            className="text-5xl md:text-7xl font-bold text-white mb-6 leading-tight tracking-tight">
            Smart Income Protection <br className="hidden md:block" /> for Delivery Partners
          </motion.h1>

          <motion.p initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ delay: 0.2 }}
            className="text-lg text-slate-300 max-w-2xl mx-auto mb-12">
            Check your live weather risk and get instant insurance payouts if severe conditions threaten your shift earnings.
          </motion.p>

          <motion.form initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} transition={{ delay: 0.3 }}
            onSubmit={handleSearch} className="max-w-2xl mx-auto relative group z-30">
            <div className="absolute inset-y-0 left-5 flex items-center pointer-events-none">
              <MapPin className="w-7 h-7 text-slate-400 group-focus-within:text-primary transition-colors" />
            </div>
            <input
              type="text"
              placeholder="Enter your delivery city (e.g., Seattle, London)..."
              className="w-full pl-16 pr-40 py-6 rounded-3xl bg-white/95 text-xl text-slate-900 placeholder:text-slate-400 shadow-2xl focus:outline-none focus:ring-4 focus:ring-primary/30 transition-all"
              value={city}
              onChange={e => setCity(e.target.value)}
            />
            <button type="submit" disabled={!city.trim() || isLoading}
              className="absolute right-3 top-3 bottom-3 px-8 bg-primary hover:bg-primary/90 text-white font-bold rounded-2xl shadow-xl transition-all disabled:opacity-50 flex items-center gap-2">
              {isLoading ? <Loader2 className="w-6 h-6 animate-spin" /> : <>Check Risk <ArrowRight className="w-5 h-5" /></>}
            </button>
          </motion.form>
        </div>
      </div>

      {/* Results */}
      <div className="max-w-5xl mx-auto px-6 -mt-20 relative z-20">
        <AnimatePresence mode="wait">

          {isLoading && (
            <motion.div key="loading" initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}
              className="grid grid-cols-1 md:grid-cols-3 gap-6">
              {[1,2,3].map(i => (
                <Card key={i} className="h-72 p-6 animate-pulse bg-white/60 backdrop-blur-md">
                  <div className="h-6 w-1/3 bg-slate-200 rounded-md mb-8" />
                  <div className="flex justify-between mb-8">
                    <div className="h-16 w-16 bg-slate-200 rounded-full" />
                    <div className="h-12 w-20 bg-slate-200 rounded-xl" />
                  </div>
                  <div className="h-4 w-full bg-slate-100 rounded-md mb-3" />
                  <div className="h-4 w-2/3 bg-slate-100 rounded-md" />
                </Card>
              ))}
            </motion.div>
          )}

          {isError && !isLoading && (
            <motion.div key="error" initial={{ opacity: 0, scale: 0.95 }} animate={{ opacity: 1, scale: 1 }} exit={{ opacity: 0 }}>
              <Card className="p-10 text-center max-w-2xl mx-auto shadow-2xl bg-white/80 backdrop-blur-xl">
                <div className="w-20 h-20 bg-rose-100 text-rose-500 rounded-full flex items-center justify-center mx-auto mb-6">
                  <MapPin className="w-10 h-10" />
                </div>
                <h3 className="text-2xl font-bold text-slate-900 mb-3">Location Not Found</h3>
                <p className="text-slate-500 text-lg mb-8">We couldn't retrieve data for "{submittedCity}". Please check the spelling and try again.</p>
                <button onClick={() => setSubmittedCity("")}
                  className="px-6 py-3 bg-slate-100 hover:bg-slate-200 text-slate-700 font-semibold rounded-xl transition-colors">
                  Clear Search
                </button>
              </Card>
            </motion.div>
          )}

          {data && !isLoading && !isError && (
            <motion.div key="results" initial={{ opacity: 0, y: 40 }} animate={{ opacity: 1, y: 0 }} className="space-y-6">

              {/* Auto Payout Banner */}
              {data.autoPayout && (
                <motion.div initial={{ opacity: 0, scale: 0.95 }} animate={{ opacity: 1, scale: 1 }}
                  className="bg-gradient-to-r from-rose-500 via-red-500 to-rose-600 rounded-3xl p-6 md:p-8 text-white shadow-2xl flex flex-col md:flex-row items-center justify-between gap-6">
                  <div className="flex items-center gap-5">
                    <div className="bg-white/20 p-4 rounded-2xl animate-pulse">
                      <ShieldAlert className="w-10 h-10" />
                    </div>
                    <div>
                      <h3 className="text-2xl font-bold mb-1">Auto Payout Triggered!</h3>
                      <p className="text-red-100 text-lg">Severe {data.weather.condition.toLowerCase()} detected in {data.city}. Your payout is ready.</p>
                    </div>
                  </div>
                  <button className="w-full md:w-auto px-8 py-4 bg-white text-red-600 font-bold text-lg rounded-2xl hover:bg-red-50 transition-all shadow-lg">
                    Claim ${data.suggestedPayout?.toFixed(2)}
                  </button>
                </motion.div>
              )}

              <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
                {/* Weather Card */}
                <Card className="p-8 bg-white/80 backdrop-blur-2xl shadow-xl hover:shadow-2xl hover:-translate-y-1 transition-all duration-300">
                  <h3 className="text-slate-500 font-semibold uppercase tracking-wider text-sm flex items-center gap-2 mb-6">
                    <Cloud className="w-5 h-5 text-slate-400" /> Current Weather
                  </h3>
                  <div className="flex items-center justify-between mb-8">
                    {getWeatherIcon(data.weather?.condition)}
                    <div className="text-right">
                      <div className="text-5xl font-bold text-slate-900">{Math.round(data.weather?.temperature || 0)}°</div>
                      <div className="text-slate-500 capitalize mt-1 text-lg">{data.weather?.description}</div>
                    </div>
                  </div>
                  <div className="grid grid-cols-2 gap-4 pt-6 border-t border-slate-100">
                    <div className="bg-slate-50 rounded-2xl p-4">
                      <div className="text-slate-400 text-xs font-semibold uppercase tracking-wider mb-2">Humidity</div>
                      <div className="font-bold text-slate-800 text-lg flex items-center gap-2">
                        <Droplets className="w-5 h-5 text-blue-400" /> {data.weather?.humidity}%
                      </div>
                    </div>
                    <div className="bg-slate-50 rounded-2xl p-4">
                      <div className="text-slate-400 text-xs font-semibold uppercase tracking-wider mb-2">Wind</div>
                      <div className="font-bold text-slate-800 text-lg flex items-center gap-2">
                        <Wind className="w-5 h-5 text-slate-400" /> {data.weather?.windSpeed}<span className="text-sm text-slate-400 font-medium">m/s</span>
                      </div>
                    </div>
                  </div>
                </Card>

                {/* Risk Card */}
                <Card className="p-8 bg-white/80 backdrop-blur-2xl shadow-xl hover:shadow-2xl hover:-translate-y-1 transition-all duration-300 flex flex-col">
                  <h3 className="text-slate-500 font-semibold uppercase tracking-wider text-sm flex items-center gap-2 mb-6">
                    <AlertTriangle className="w-5 h-5 text-slate-400" /> Risk Assessment
                  </h3>
                  <div className="flex-1 flex flex-col items-center justify-center py-4">
                    <span className={cn("px-5 py-2 rounded-full text-sm font-bold border shadow-sm flex items-center gap-2 mb-6",
                      riskTheme[data.riskLevel as keyof typeof riskTheme]?.badge)}>
                      {riskTheme[data.riskLevel as keyof typeof riskTheme]?.icon}
                      {data.riskLevel} Risk
                    </span>
                    <div className="relative">
                      <svg className="w-32 h-32 transform -rotate-90">
                        <circle cx="64" cy="64" r="56" className="stroke-slate-100" strokeWidth="12" fill="none" />
                        <circle cx="64" cy="64" r="56"
                          className={riskTheme[data.riskLevel as keyof typeof riskTheme]?.color}
                          stroke="currentColor" strokeWidth="12" fill="none"
                          strokeDasharray="351.8"
                          strokeDashoffset={351.8 - (351.8 * (data.risk?.riskScore || 0)) / 100}
                          strokeLinecap="round" />
                      </svg>
                      <div className="absolute inset-0 flex items-center justify-center">
                        <span className="text-4xl font-bold text-slate-900">{data.risk?.riskScore}</span>
                      </div>
                    </div>
                  </div>
                  <div className="pt-6 border-t border-slate-100 text-center text-slate-600 text-sm leading-relaxed font-medium">
                    "{data.risk?.reason}"
                  </div>
                </Card>

                {/* Coverage Card */}
                <Card className="p-8 bg-gradient-to-br from-slate-900 via-slate-800 to-slate-900 text-white border-0 shadow-2xl hover:-translate-y-1 transition-all duration-300 relative overflow-hidden">
                  <div className="absolute -right-10 -top-10 w-40 h-40 bg-primary/20 rounded-full blur-3xl pointer-events-none" />
                  <h3 className="text-slate-400 font-semibold uppercase tracking-wider text-sm flex items-center gap-2 mb-8">
                    <ShieldCheck className="w-5 h-5 text-teal-400" /> Coverage Details
                  </h3>
                  <div className="space-y-8 flex flex-col h-[calc(100%-4rem)]">
                    <div>
                      <div className="text-slate-400 text-sm font-medium mb-3">Estimated Income Loss</div>
                      <div className="text-4xl font-bold text-slate-200 flex items-center gap-3">
                        <TrendingDown className="w-8 h-8 text-rose-400/80" />
                        ${data.estimatedIncomeLoss?.toFixed(2)}
                      </div>
                    </div>
                    <div className="pt-8 border-t border-slate-700/50 mt-auto">
                      <div className="text-slate-400 text-sm font-medium mb-3 flex items-center justify-between">
                        Suggested Payout
                        <span className="text-xs bg-teal-500/20 text-teal-300 px-2 py-1 rounded-md font-bold">READY</span>
                      </div>
                      <div className="text-5xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-teal-300 to-emerald-300">
                        ${data.suggestedPayout?.toFixed(2)}
                      </div>
                    </div>
                  </div>
                </Card>
              </div>
            </motion.div>
          )}
        </AnimatePresence>
      </div>

      <footer className="mt-32 pb-8 text-center text-slate-400 text-sm font-medium">
        <p>© {new Date().getFullYear()} GigSecure AI. Designed for modern delivery partners.</p>
      </footer>
    </div>
  );
}

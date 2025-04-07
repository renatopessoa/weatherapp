# Roadmap para Implementação do Dashboard de Clima

## 1. Configuração Inicial do Projeto

### 1.1. Criação do Projeto Next.js
```bash
# Criar um novo projeto Next.js com TypeScript
npx create-next-app@latest weather-dashboard --typescript
cd weather-dashboard

# Instalar dependências do Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 1.2. Configuração do Tailwind CSS
Editar o arquivo `tailwind.config.js`:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  darkMode: 'class', // Habilitar modo escuro
  theme: {
    extend: {
      colors: {
        'weather-primary': '#1E213A',
        'weather-secondary': '#100E1D',
        'weather-accent': '#3C47E9',
        'weather-text': '#E7E7EB',
        'weather-text-secondary': '#A09FB1',
      },
      fontFamily: {
        sans: ['Inter', 'sans-serif'],
      },
      backgroundImage: {
        'gradient-radial': 'radial-gradient(var(--tw-gradient-stops))',
        'weather-pattern': "url('/images/weather-pattern.svg')",
      },
    },
  },
  plugins: [],
}
```

### 1.3. Configuração do CSS Global
Editar o arquivo `globals.css`:
```css
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');

@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 0, 0, 0;
  --background-start-rgb: 214, 219, 220;
  --background-end-rgb: 255, 255, 255;
}

.dark {
  --foreground-rgb: 255, 255, 255;
  --background-start-rgb: 30, 33, 58;
  --background-end-rgb: 16, 14, 29;
}

body {
  color: rgb(var(--foreground-rgb));
  background: linear-gradient(
      to bottom,
      transparent,
      rgb(var(--background-end-rgb))
    )
    rgb(var(--background-start-rgb));
}

@layer components {
  .weather-card {
    @apply bg-white/10 backdrop-blur-md rounded-2xl p-6 shadow-lg dark:bg-weather-primary;
  }
  
  .weather-input {
    @apply bg-white/10 dark:bg-weather-secondary border-0 rounded-full px-4 py-2 text-sm focus:ring-2 focus:ring-weather-accent focus:outline-none;
  }
}
```

### 1.4. Instalação de Bibliotecas Adicionais
```bash
# Bibliotecas para gráficos e visualizações
npm install chart.js react-chartjs-2

# Biblioteca para ícones
npm install react-icons

# Biblioteca para requisições HTTP
npm install axios

# Biblioteca para manipulação de datas
npm install date-fns

# Biblioteca para animações
npm install framer-motion

# Biblioteca para gerenciamento de estado
npm install zustand
```

## 2. Estrutura de Pastas e Arquivos

### 2.1. Estrutura de Diretórios
```
weather-dashboard/
├── app/
│   ├── api/
│   │   └── weather/
│   │       └── route.ts
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Toggle.tsx
│   │   ├── weather/
│   │   │   ├── CurrentWeather.tsx
│   │   │   ├── ForecastChart.tsx
│   │   │   ├── SearchBar.tsx
│   │   │   ├── WeatherAlert.tsx
│   │   │   ├── WeatherDetails.tsx
│   │   │   ├── WeatherForecast.tsx
│   │   │   └── CityComparison.tsx
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   └── Footer.tsx
│   │   └── theme/
│   │       └── ThemeToggle.tsx
│   ├── context/
│   │   ├── ThemeContext.tsx
│   │   └── WeatherContext.tsx
│   ├── hooks/
│   │   ├── useWeather.ts
│   │   ├── useTheme.ts
│   │   └── useLocalStorage.ts
│   ├── lib/
│   │   ├── api.ts
│   │   └── utils.ts
│   ├── store/
│   │   └── weatherStore.ts
│   ├── types/
│   │   ├── weather.ts
│   │   └── api.ts
│   ├── favicon.ico
│   ├── globals.css
│   ├── layout.tsx
│   └── page.tsx
├── public/
│   ├── images/
│   │   ├── weather-icons/
│   │   │   ├── clear.svg
│   │   │   ├── clouds.svg
│   │   │   ├── rain.svg
│   │   │   ├── snow.svg
│   │   │   └── ...
│   │   ├── weather-pattern.svg
│   │   └── background.jpg
│   └── fonts/
├── .env.local
├── next.config.js
├── package.json
├── tailwind.config.js
└── tsconfig.json
```

### 2.2. Definição de Tipos
Criar arquivo `types/weather.ts`:
```typescript
export interface WeatherData {
  city: string;
  country: string;
  temperature: number;
  feels_like: number;
  humidity: number;
  wind_speed: number;
  wind_direction: string;
  weather_condition: string;
  weather_description: string;
  weather_icon: string;
  precipitation: number;
  date: string;
  time: string;
}

export interface ForecastData {
  date: string;
  time: string;
  temperature: number;
  feels_like: number;
  humidity: number;
  wind_speed: number;
  weather_condition: string;
  weather_icon: string;
  precipitation: number;
}

export interface WeatherAlert {
  id: string;
  type: string;
  severity: 'low' | 'medium' | 'high';
  title: string;
  description: string;
  start: string;
  end: string;
}

export interface CityComparisonData {
  city: string;
  temperature: number;
  color: string;
}

export interface AirQualityData {
  index: number;
  description: string;
  components: {
    co: number;
    no2: number;
    o3: number;
    pm2_5: number;
    pm10: number;
    so2: number;
  }
}
```

Criar arquivo `types/api.ts`:
```typescript
export interface OpenWeatherMapResponse {
  coord: {
    lon: number;
    lat: number;
  };
  weather: Array<{
    id: number;
    main: string;
    description: string;
    icon: string;
  }>;
  base: string;
  main: {
    temp: number;
    feels_like: number;
    temp_min: number;
    temp_max: number;
    pressure: number;
    humidity: number;
    sea_level?: number;
    grnd_level?: number;
  };
  visibility: number;
  wind: {
    speed: number;
    deg: number;
    gust?: number;
  };
  rain?: {
    '1h'?: number;
    '3h'?: number;
  };
  snow?: {
    '1h'?: number;
    '3h'?: number;
  };
  clouds: {
    all: number;
  };
  dt: number;
  sys: {
    type: number;
    id: number;
    country: string;
    sunrise: number;
    sunset: number;
  };
  timezone: number;
  id: number;
  name: string;
  cod: number;
}

export interface OpenWeatherMapForecastResponse {
  cod: string;
  message: number;
  cnt: number;
  list: Array<{
    dt: number;
    main: {
      temp: number;
      feels_like: number;
      temp_min: number;
      temp_max: number;
      pressure: number;
      sea_level: number;
      grnd_level: number;
      humidity: number;
      temp_kf: number;
    };
    weather: Array<{
      id: number;
      main: string;
      description: string;
      icon: string;
    }>;
    clouds: {
      all: number;
    };
    wind: {
      speed: number;
      deg: number;
      gust?: number;
    };
    visibility: number;
    pop: number;
    rain?: {
      '3h': number;
    };
    snow?: {
      '3h': number;
    };
    sys: {
      pod: string;
    };
    dt_txt: string;
  }>;
  city: {
    id: number;
    name: string;
    coord: {
      lat: number;
      lon: number;
    };
    country: string;
    population: number;
    timezone: number;
    sunrise: number;
    sunset: number;
  };
}

export interface OpenWeatherMapAirPollutionResponse {
  coord: {
    lon: number;
    lat: number;
  };
  list: Array<{
    main: {
      aqi: number;
    };
    components: {
      co: number;
      no: number;
      no2: number;
      o3: number;
      so2: number;
      pm2_5: number;
      pm10: number;
      nh3: number;
    };
    dt: number;
  }>;
}

export interface OpenWeatherMapAlertsResponse {
  alerts: Array<{
    sender_name: string;
    event: string;
    start: number;
    end: number;
    description: string;
    tags: string[];
  }>;
}
```

## 3. Configuração da API e Serviços

### 3.1. Configuração das Variáveis de Ambiente
Criar arquivo `.env.local`:
```
OPENWEATHERMAP_API_KEY=sua_chave_api_aqui
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

### 3.2. Configuração do Serviço de API
Criar arquivo `lib/api.ts`:
```typescript
import axios from 'axios';
import { 
  OpenWeatherMapResponse, 
  OpenWeatherMapForecastResponse,
  OpenWeatherMapAirPollutionResponse,
  OpenWeatherMapAlertsResponse
} from '../types/api';

const API_KEY = process.env.OPENWEATHERMAP_API_KEY;
const BASE_URL = 'https://api.openweathermap.org/data/2.5';

export const fetchWeatherData = async (city: string): Promise<OpenWeatherMapResponse> => {
  try {
    const response = await axios.get(`${BASE_URL}/weather`, {
      params: {
        q: city,
        appid: API_KEY,
        units: 'metric',
        lang: 'pt_br'
      }
    });
    return response.data;
  } catch (error) {
    console.error('Erro ao buscar dados do clima:', error);
    throw error;
  }
};

export const fetchForecastData = async (city: string): Promise<OpenWeatherMapForecastResponse> => {
  try {
    const response = await axios.get(`${BASE_URL}/forecast`, {
      params: {
        q: city,
        appid: API_KEY,
        units: 'metric',
        lang: 'pt_br'
      }
    });
    return response.data;
  } catch (error) {
    console.error('Erro ao buscar previsão do tempo:', error);
    throw error;
  }
};

export const fetchAirPollutionData = async (lat: number, lon: number): Promise<OpenWeatherMapAirPollutionResponse> => {
  try {
    const response = await axios.get(`${BASE_URL}/air_pollution`, {
      params: {
        lat,
        lon,
        appid: API_KEY
      }
    });
    return response.data;
  } catch (error) {
    console.error('Erro ao buscar dados de poluição do ar:', error);
    throw error;
  }
};

export const fetchWeatherAlerts = async (lat: number, lon: number): Promise<OpenWeatherMapAlertsResponse> => {
  try {
    const response = await axios.get(`${BASE_URL}/onecall`, {
      params: {
        lat,
        lon,
        appid: API_KEY,
        exclude: 'current,minutely,hourly,daily',
        lang: 'pt_br'
      }
    });
    return response.data;
  } catch (error) {
    console.error('Erro ao buscar alertas meteorológicos:', error);
    throw error;
  }
};
```

### 3.3. Criação da API Route para o Next.js
Criar arquivo `app/api/weather/route.ts`:
```typescript
import { NextResponse } from 'next/server';
import { 
  fetchWeatherData, 
  fetchForecastData, 
  fetchAirPollutionData,
  fetchWeatherAlerts
} from '@/app/lib/api';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const city = searchParams.get('city');
  
  if (!city) {
    return NextResponse.json({ error: 'Parâmetro de cidade é obrigatório' }, { status: 400 });
  }
  
  try {
    // Buscar dados atuais do clima
    const weatherData = await fetchWeatherData(city);
    
    // Extrair coordenadas para outras chamadas de API
    const { lat, lon } = weatherData.coord;
    
    // Buscar previsão do tempo
    const forecastData = await fetchForecastData(city);
    
    // Buscar dados de qualidade do ar
    const airPollutionData = await fetchAirPollutionData(lat, lon);
    
    // Buscar alertas meteorológicos
    const alertsData = await fetchWeatherAlerts(lat, lon);
    
    return NextResponse.json({
      current: weatherData,
      forecast: forecastData,
      airQuality: airPollutionData,
      alerts: alertsData
    });
  } catch (error) {
    console.error('Erro ao processar requisição de clima:', error);
    return NextResponse.json({ error: 'Falha ao buscar dados meteorológicos' }, { status: 500 });
  }
}
```

## 4. Gerenciamento de Estado

### 4.1. Criação do Store com Zustand
Criar arquivo `store/weatherStore.ts`:
```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { 
  WeatherData, 
  ForecastData, 
  WeatherAlert, 
  AirQualityData,
  CityComparisonData
} from '../types/weather';

interface WeatherState {
  currentWeather: WeatherData | null;
  forecast: ForecastData[];
  alerts: WeatherAlert[];
  airQuality: AirQualityData | null;
  cityComparison: CityComparisonData[];
  searchHistory: string[];
  isLoading: boolean;
  error: string | null;
  selectedCity: string;
  
  // Ações
  setCurrentWeather: (data: WeatherData) => void;
  setForecast: (data: ForecastData[]) => void;
  setAlerts: (data: WeatherAlert[]) => void;
  setAirQuality: (data: AirQualityData) => void;
  setCityComparison: (data: CityComparisonData[]) => void;
  addToSearchHistory: (city: string) => void;
  setLoading: (isLoading: boolean) => void;
  setError: (error: string | null) => void;
  setSelectedCity: (city: string) => void;
  fetchWeatherData: (city: string) => Promise<void>;
}

export const useWeatherStore = create<WeatherState>()(
  persist(
    (set, get) => ({
      currentWeather: null,
      forecast: [],
      alerts: [],
      airQuality: null,
      cityComparison: [],
      searchHistory: [],
      isLoading: false,
      error: null,
      selectedCity: 'São Paulo',
      
      setCurrentWeather: (data) => set({ currentWeather: data }),
      setForecast: (data) => set({ forecast: data }),
      setAlerts: (data) => set({ alerts: data }),
      setAirQuality: (data) => set({ airQuality: data }),
      setCityComparison: (data) => set({ cityComparison: data }),
      
      addToSearchHistory: (city) => {
        const history = get().searchHistory;
        if (!history.includes(city)) {
          set({ searchHistory: [city, ...history].slice(0, 5) });
        }
      },
      
      setLoading: (isLoading) => set({ isLoading }),
      setError: (error) => set({ error }),
      setSelectedCity: (city) => set({ selectedCity: city }),
      
      fetchWeatherData: async (city) => {
        const { addToSearchHistory, setLoading, setError } = get();
        
        try {
          setLoading(true);
          setError(null);
          
          const response = await fetch(`/api/weather?city=${encodeURIComponent(city)}`);
          
          if (!response.ok) {
            const errorData = await response.json();
            throw new Error(errorData.error || 'Falha ao buscar dados meteorológicos');
          }
          
          const data = await response.json();
          
          // Processar e transformar dados da API para o formato interno
          // Implementar lógica de transformação aqui
          
          // Adicionar cidade ao histórico de pesquisa
          addToSearchHistory(city);
          
          setLoading(false);
        } catch (error) {
          setLoading(false);
          setError(error instanceof Error ? error.message : 'Erro desconhecido');
        }
      }
    }),
    {
      name: 'weather-storage',
      partialize: (state) => ({ 
        searchHistory: state.searchHistory,
        selectedCity: state.selectedCity
      })
    }
  )
);
```

### 4.2. Criação do Contexto de Tema
Criar arquivo `context/ThemeContext.tsx`:
```typescript
'use client';

import React, { createContext, useContext, useState, useEffect } from 'react';

type Theme = 'light' | 'dark';

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

export const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [theme, setTheme] = useState<Theme>('dark');

  useEffect(() => {
    // Verificar preferência salva no localStorage
    const savedTheme = localStorage.getItem('theme') as Theme | null;
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    
    if (savedTheme) {
      setTheme(savedTheme);
      document.documentElement.classList.toggle('dark', savedTheme === 'dark');
    } else if (prefersDark) {
      setTheme('dark');
      document.documentElement.classList.add('dark');
    }
  }, []);

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';
    setTheme(newTheme);
    document.documentElement.classList.toggle('dark', newTheme === 'dark');
    localStorage.setItem('theme', newTheme);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (context === undefined) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
};
```

## 5. Implementação dos Componentes

### 5.1. Layout Principal
Criar arquivo `app/layout.tsx`:
```tsx
import './globals.css';
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import { ThemeProvider } from './context/ThemeContext';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'Weather Dashboard',
  description: 'Dashboard de clima com dados em tempo real',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="pt-BR">
      <body className={inter.className}>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 5.2. Página Principal
Criar arquivo `app/page.tsx`:
```tsx
'use client';

import { useEffect } from 'react';
import { useWeatherStore } from './store/weatherStore';
import Header from './components/layout/Header';
import Sidebar from './components/layout/Sidebar';
import CurrentWeather from './components/weather/CurrentWeather';
import WeatherForecast from './components/weather/WeatherForecast';
import ForecastChart from './components/weather/ForecastChart';
import WeatherDetails from './components/weather/WeatherDetails';
import CityComparison from './components/weather/CityComparison';
import WeatherAlert from './components/weather/WeatherAlert';

export default function Home() {
  const { fetchWeatherData, selectedCity, isLoading, error } = useWeatherStore();

  useEffect(() => {
    fetchWeatherData(selectedCity);
  }, [fetchWeatherData, selectedCity]);

  return (
    <main className="min-h-screen bg-gradient-to-b from-gray-100 to-gray-200 dark:from-weather-primary dark:to-weather-secondary">
      <Header />
      
      <div className="container mx-auto px-4 py-8 flex flex-col lg:flex-row gap-6">
        <Sidebar />
        
        <div className="flex-1 space-y-6">
          {isLoading ? (
            <div className="weather-card flex items-center justify-center h-64">
              <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-weather-accent"></div>
            </div>
          ) : error ? (
            <div className="weather-card bg-red-100 dark:bg-red-900/20 text-red-800 dark:text-red-200 p-4">
              <p>{error}</p>
            </div>
          ) : (
            <>
              <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div className="lg:col-span-2">
                  <CurrentWeather />
                </div>
                <div>
                  <WeatherAlert />
                </div>
              </div>
              
              <div className="weather-card">
                <h2 className="text-xl font-semibold mb-4">Previsão do Tempo</h2>
                <WeatherForecast />
              </div>
              
              <div className="weather-card">
                <h2 className="text-xl font-semibold mb-4">Variação de Temperatura</h2>
                <ForecastChart />
              </div>
              
              <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
                <div className="weather-card">
                  <h2 className="text-xl font-semibold mb-4">Detalhes</h2>
                  <WeatherDetails />
                </div>
                <div className="weather-card">
                  <h2 className="text-xl font-semibold mb-4">Comparação de Cidades</h2>
                  <CityComparison />
                </div>
              </div>
            </>
          )}
        </div>
      </div>
    </main>
  );
}
```

### 5.3. Componentes de UI
Criar arquivo `components/ui/Button.tsx`:
```tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  onClick?: () => void;
  className?: string;
  disabled?: boolean;
  type?: 'button' | 'submit' | 'reset';
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  onClick,
  className = '',
  disabled = false,
  type = 'button',
}) => {
  const baseClasses = 'rounded-full font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-weather-accent text-white hover:bg-blue-700 focus:ring-blue-500',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600 focus:ring-gray-500',
    outline: 'bg-transparent border border-gray-300 dark:border-gray-600 text-gray-700 dark:text-gray-300 hover:bg-gray-100 dark:hover:bg-gray-800 focus:ring-gray-500',
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-xs',
    md: 'px-4 py-2 text-sm',
    lg: 'px-6 py-3 text-base',
  };
  
  const disabledClasses = disabled ? 'opacity-50 cursor-not-allowed' : 'cursor-pointer';
  
  return (
    <button
      type={type}
      className={`${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${disabledClasses} ${className}`}
      onClick={onClick}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;
```

Criar arquivo `components/ui/Card.tsx`:
```tsx
import React from 'react';

interface CardProps {
  children: React.ReactNode;
  className?: string;
}

const Card: React.FC<CardProps> = ({ children, className = '' }) => {
  return (
    <div className={`weather-card ${className}`}>
      {children}
    </div>
  );
};

export default Card;
```

Criar arquivo `components/ui/Input.tsx`:
```tsx
import React from 'react';

interface InputProps {
  placeholder?: string;
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onKeyDown?: (e: React.KeyboardEvent<HTMLInputElement>) => void;
  className?: string;
  icon?: React.ReactNode;
}

const Input: React.FC<InputProps> = ({
  placeholder,
  value,
  onChange,
  onKeyDown,
  className = '',
  icon,
}) => {
  return (
    <div className="relative">
      {icon && (
        <div className="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none text-gray-400 dark:text-gray-500">
          {icon}
        </div>
      )}
      <input
        type="text"
        className={`weather-input w-full ${icon ? 'pl-10' : 'pl-4'} ${className}`}
        placeholder={placeholder}
        value={value}
        onChange={onChange}
        onKeyDown={onKeyDown}
      />
    </div>
  );
};

export default Input;
```

Criar arquivo `components/ui/Toggle.tsx`:
```tsx
import React from 'react';

interface ToggleProps {
  enabled: boolean;
  onChange: () => void;
  label?: string;
  className?: string;
}

const Toggle: React.FC<ToggleProps> = ({
  enabled,
  onChange,
  label,
  className = '',
}) => {
  return (
    <div className={`flex items-center ${className}`}>
      {label && (
        <span className="mr-3 text-sm font-medium text-gray-700 dark:text-gray-300">
          {label}
        </span>
      )}
      <button
        type="button"
        className={`relative inline-flex h-6 w-11 items-center rounded-full focus:outline-none focus:ring-2 focus:ring-weather-accent focus:ring-offset-2 ${
          enabled ? 'bg-weather-accent' : 'bg-gray-200 dark:bg-gray-700'
        }`}
        onClick={onChange}
      >
        <span
          className={`inline-block h-4 w-4 transform rounded-full bg-white transition ${
            enabled ? 'translate-x-6' : 'translate-x-1'
          }`}
        />
      </button>
    </div>
  );
};

export default Toggle;
```

### 5.4. Componentes de Layout
Criar arquivo `components/layout/Header.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { useTheme } from '@/app/context/ThemeContext';
import { FiSun, FiMoon, FiSearch } from 'react-icons/fi';
import { useWeatherStore } from '@/app/store/weatherStore';
import Input from '../ui/Input';

const Header: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  const { fetchWeatherData, setSelectedCity } = useWeatherStore();
  const [searchQuery, setSearchQuery] = useState('');

  const handleSearch = () => {
    if (searchQuery.trim()) {
      setSelectedCity(searchQuery);
      fetchWeatherData(searchQuery);
      setSearchQuery('');
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      handleSearch();
    }
  };

  return (
    <header className="bg-white/80 dark:bg-weather-primary/80 backdrop-blur-md shadow-sm sticky top-0 z-10">
      <div className="container mx-auto px-4 py-4 flex items-center justify-between">
        <div className="flex items-center">
          <h1 className="text-xl font-bold text-gray-800 dark:text-white">Weather Dashboard</h1>
          <span className="ml-4 text-sm text-gray-500 dark:text-gray-400">2023</span>
        </div>
        
        <div className="flex items-center space-x-4">
          <div className="w-64">
            <Input
              placeholder="Buscar cidade..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              onKeyDown={handleKeyDown}
              icon={<FiSearch />}
            />
          </div>
          
          <button
            onClick={toggleTheme}
            className="p-2 rounded-full hover:bg-gray-200 dark:hover:bg-gray-800 transition-colors"
            aria-label={theme === 'dark' ? 'Ativar modo claro' : 'Ativar modo escuro'}
          >
            {theme === 'dark' ? <FiSun className="text-yellow-400" /> : <FiMoon className="text-gray-600" />}
          </button>
        </div>
      </div>
    </header>
  );
};

export default Header;
```

Criar arquivo `components/layout/Sidebar.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { useWeatherStore } from '@/app/store/weatherStore';
import { FiClock, FiMapPin } from 'react-icons/fi';
import Button from '../ui/Button';

const Sidebar: React.FC = () => {
  const { searchHistory, fetchWeatherData, setSelectedCity, currentWeather } = useWeatherStore();
  const [showHistory, setShowHistory] = useState(false);

  const handleCitySelect = (city: string) => {
    setSelectedCity(city);
    fetchWeatherData(city);
    setShowHistory(false);
  };

  return (
    <div className="lg:w-80 space-y-6">
      <div className="weather-card">
        <div className="flex items-center space-x-2 text-2xl font-light mb-6">
          <FiMapPin className="text-weather-accent" />
          <h2>{currentWeather?.city || 'Carregando...'}</h2>
        </div>
        
        <div className="text-7xl font-light mb-2">
          {currentWeather ? `${Math.round(currentWeather.temperature)}°` : '--°'}
        </div>
        
        <div className="text-xl text-gray-600 dark:text-gray-300 mb-4">
          {currentWeather?.weather_description || 'Carregando...'}
        </div>
        
        <div className="flex justify-between text-sm text-gray-500 dark:text-gray-400">
          <div>
            Vento: {currentWeather ? `${currentWeather.wind_speed} km/h` : '--'}
          </div>
          <div>
            Umidade: {currentWeather ? `${currentWeather.humidity}%` : '--'}
          </div>
        </div>
      </div>
      
      <div className="weather-card">
        <div className="flex items-center justify-between mb-4">
          <div className="flex items-center space-x-2">
            <FiClock className="text-weather-accent" />
            <h3 className="font-medium">Pesquisas Recentes</h3>
          </div>
          <Button
            variant="outline"
            size="sm"
            onClick={() => setShowHistory(!showHistory)}
          >
            {showHistory ? 'Ocultar' : 'Mostrar'}
          </Button>
        </div>
        
        {showHistory && (
          <div className="space-y-2">
            {searchHistory.length > 0 ? (
              searchHistory.map((city, index) => (
                <div
                  key={`${city}-${index}`}
                  className="p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-800 cursor-pointer transition-colors"
                  onClick={() => handleCitySelect(city)}
                >
                  {city}
                </div>
              ))
            ) : (
              <p className="text-sm text-gray-500 dark:text-gray-400">
                Nenhuma pesquisa recente
              </p>
            )}
          </div>
        )}
      </div>
    </div>
  );
};

export default Sidebar;
```

### 5.5. Componentes de Clima
Criar arquivo `components/weather/CurrentWeather.tsx`:
```tsx
'use client';

import { useWeatherStore } from '@/app/store/weatherStore';
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';

const CurrentWeather: React.FC = () => {
  const { currentWeather } = useWeatherStore();

  if (!currentWeather) {
    return (
      <div className="weather-card h-64 flex items-center justify-center">
        <p>Carregando dados do clima...</p>
      </div>
    );
  }

  return (
    <div className="weather-card h-full relative overflow-hidden">
      <div className="absolute top-0 right-0 p-4">
        <div className="text-sm text-gray-500 dark:text-gray-400">
          NATIONAL<br />WEATHER
        </div>
      </div>
      
      <div className="mb-6">
        <h2 className="text-lg font-medium text-gray-600 dark:text-gray-300">
          Weather Forecast
        </h2>
        
        <div className="text-4xl font-bold mt-2">
          {currentWeather.weather_condition}
          <span className="block text-5xl mt-1">
            with {currentWeather.weather_description}
          </span>
        </div>
      </div>
      
      <div className="mb-4">
        <div className="flex items-center text-sm text-gray-600 dark:text-gray-300">
          <span>{currentWeather.country}, </span>
          <span className="ml-1">
            {format(new Date(currentWeather.date), "EEEE, MMM d, yyyy, h:mm aaa", { locale: ptBR })}
          </span>
        </div>
      </div>
      
      <div className="text-sm text-gray-600 dark:text-gray-300">
        <p>
          {currentWeather.weather_description}. 
          Vento: {currentWeather.wind_speed} km/h. 
          Chance de chuva: {currentWeather.precipitation}%.
        </p>
      </div>
    </div>
  );
};

export default CurrentWeather;
```

Criar arquivo `components/weather/ForecastChart.tsx`:
```tsx
'use client';

import { useEffect, useRef } from 'react';
import { useWeatherStore } from '@/app/store/weatherStore';
import { Chart, registerables } from 'chart.js';
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';

// Registrar componentes do Chart.js
Chart.register(...registerables);

const ForecastChart: React.FC = () => {
  const { forecast } = useWeatherStore();
  const chartRef = useRef<HTMLCanvasElement>(null);
  const chartInstance = useRef<Chart | null>(null);

  useEffect(() => {
    if (!forecast.length || !chartRef.current) return;

    // Destruir gráfico anterior se existir
    if (chartInstance.current) {
      chartInstance.current.destroy();
    }

    const ctx = chartRef.current.getContext('2d');
    if (!ctx) return;

    // Preparar dados para o gráfico
    const labels = forecast.slice(0, 8).map(item => 
      format(new Date(item.date + ' ' + item.time), 'HH:mm', { locale: ptBR })
    );
    
    const temperatures = forecast.slice(0, 8).map(item => item.temperature);

    // Criar novo gráfico
    chartInstance.current = new Chart(ctx, {
      type: 'line',
      data: {
        labels,
        datasets: [
          {
            label: 'Temperatura (°C)',
            data: temperatures,
            borderColor: '#3C47E9',
            backgroundColor: 'rgba(60, 71, 233, 0.1)',
            tension: 0.4,
            fill: true,
            pointBackgroundColor: '#3C47E9',
            pointRadius: 4,
            pointHoverRadius: 6,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: {
            display: false,
          },
          tooltip: {
            mode: 'index',
            intersect: false,
            backgroundColor: 'rgba(30, 33, 58, 0.8)',
            titleColor: '#E7E7EB',
            bodyColor: '#E7E7EB',
            borderColor: 'rgba(255, 255, 255, 0.1)',
            borderWidth: 1,
          },
        },
        scales: {
          x: {
            grid: {
              display: false,
            },
            ticks: {
              color: '#A09FB1',
            },
          },
          y: {
            grid: {
              color: 'rgba(160, 159, 177, 0.1)',
            },
            ticks: {
              color: '#A09FB1',
            },
          },
        },
      },
    });

    return () => {
      if (chartInstance.current) {
        chartInstance.current.destroy();
      }
    };
  }, [forecast]);

  return (
    <div className="h-64">
      {forecast.length > 0 ? (
        <canvas ref={chartRef} />
      ) : (
        <div className="h-full flex items-center justify-center">
          <p className="text-gray-500 dark:text-gray-400">
            Dados de previsão não disponíveis
          </p>
        </div>
      )}
    </div>
  );
};

export default ForecastChart;
```

Criar arquivo `components/weather/SearchBar.tsx`:
```tsx
'use client';

import { useState } from 'react';
import { FiSearch } from 'react-icons/fi';
import { useWeatherStore } from '@/app/store/weatherStore';
import Input from '../ui/Input';
import Button from '../ui/Button';

const SearchBar: React.FC = () => {
  const { fetchWeatherData, setSelectedCity } = useWeatherStore();
  const [searchQuery, setSearchQuery] = useState('');

  const handleSearch = () => {
    if (searchQuery.trim()) {
      setSelectedCity(searchQuery);
      fetchWeatherData(searchQuery);
      setSearchQuery('');
    }
  };

  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      handleSearch();
    }
  };

  return (
    <div className="flex space-x-2">
      <Input
        placeholder="Buscar cidade..."
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        onKeyDown={handleKeyDown}
        icon={<FiSearch />}
        className="flex-1"
      />
      <Button onClick={handleSearch}>
        Buscar
      </Button>
    </div>
  );
};

export default SearchBar;
```

Criar arquivo `components/weather/WeatherAlert.tsx`:
```tsx
'use client';

import { useWeatherStore } from '@/app/store/weatherStore';
import { FiAlertTriangle } from 'react-icons/fi';

const WeatherAlert: React.FC = () => {
  const { alerts } = useWeatherStore();

  if (!alerts || alerts.length === 0) {
    return (
      <div className="weather-card h-full flex flex-col justify-center items-center text-center p-6">
        <div className="text-gray-400 dark:text-gray-500 mb-2">
          <FiAlertTriangle size={24} />
        </div>
        <p className="text-gray-500 dark:text-gray-400">
          Nenhum alerta meteorológico ativo para esta região.
        </p>
      </div>
    );
  }

  // Exibir apenas o alerta mais crítico
  const criticalAlert = alerts.sort((a, b) => {
    const severityOrder = { high: 3, medium: 2, low: 1 };
    return severityOrder[b.severity] - severityOrder[a.severity];
  })[0];

  const severityColor = {
    high: 'text-red-500',
    medium: 'text-orange-500',
    low: 'text-yellow-500',
  };

  return (
    <div className="weather-card h-full">
      <div className="flex items-center space-x-2 mb-4">
        <FiAlertTriangle className={severityColor[criticalAlert.severity]} />
        <h3 className={`font-medium ${severityColor[criticalAlert.severity]}`}>
          {criticalAlert.title}
        </h3>
      </div>
      
      <p className="text-sm text-gray-600 dark:text-gray-300 mb-4">
        {criticalAlert.description}
      </p>
      
      <div className="text-xs text-gray-500 dark:text-gray-400">
        <p>Início: {new Date(criticalAlert.start).toLocaleString()}</p>
        <p>Fim: {new Date(criticalAlert.end).toLocaleString()}</p>
      </div>
    </div>
  );
};

export default WeatherAlert;
```

Criar arquivo `components/weather/WeatherDetails.tsx`:
```tsx
'use client';

import { useWeatherStore } from '@/app/store/weatherStore';
import { FiDroplet, FiWind, FiSun, FiThermometer } from 'react-icons/fi';

const WeatherDetails: React.FC = () => {
  const { currentWeather, airQuality } = useWeatherStore();

  if (!currentWeather) {
    return (
      <div className="flex items-center justify-center h-40">
        <p className="text-gray-500 dark:text-gray-400">
          Carregando detalhes...
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="grid grid-cols-2 gap-4">
        <div className="flex items-center space-x-3">
          <div className="p-2 bg-blue-100 dark:bg-blue-900/20 rounded-full text-blue-500">
            <FiDroplet />
          </div>
          <div>
            <p className="text-sm text-gray-500 dark:text-gray-400">Umidade</p>
            <p className="font-medium">{currentWeather.humidity}%</p>
          </div>
        </div>
        
        <div className="flex items-center space-x-3">
          <div className="p-2 bg-green-100 dark:bg-green-900/20 rounded-full text-green-500">
            <FiWind />
          </div>
          <div>
            <p className="text-sm text-gray-500 dark:text-gray-400">Vento</p>
            <p className="font-medium">{currentWeather.wind_speed} km/h</p>
          </div>
        </div>
        
        <div className="flex items-center space-x-3">
          <div className="p-2 bg-yellow-100 dark:bg-yellow-900/20 rounded-full text-yellow-500">
            <FiSun />
          </div>
          <div>
            <p className="text-sm text-gray-500 dark:text-gray-400">Precipitação</p>
            <p className="font-medium">{currentWeather.precipitation}%</p>
          </div>
        </div>
        
        <div className="flex items-center space-x-3">
          <div className="p-2 bg-red-100 dark:bg-red-900/20 rounded-full text-red-500">
            <FiThermometer />
          </div>
          <div>
            <p className="text-sm text-gray-500 dark:text-gray-400">Sensação</p>
            <p className="font-medium">{Math.round(currentWeather.feels_like)}°C</p>
          </div>
        </div>
      </div>
      
      {airQuality && (
        <div>
          <h3 className="text-sm font-medium text-gray-600 dark:text-gray-300 mb-2">
            Qualidade do Ar
          </h3>
          <p className="text-sm text-gray-500 dark:text-gray-400 mb-1">
            {airQuality.description}
          </p>
          <div className="h-2 bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
            <div 
              className="h-full bg-gradient-to-r from-green-500 to-red-500"
              style={{ width: `${(airQuality.index / 5) * 100}%` }}
            ></div>
          </div>
        </div>
      )}
    </div>
  );
};

export default WeatherDetails;
```

Criar arquivo `components/weather/WeatherForecast.tsx`:
```tsx
'use client';

import { useWeatherStore } from '@/app/store/weatherStore';
import { format } from 'date-fns';
import { ptBR } from 'date-fns/locale';

const WeatherForecast: React.FC = () => {
  const { forecast } = useWeatherStore();

  if (!forecast || forecast.length === 0) {
    return (
      <div className="flex items-center justify-center h-40">
        <p className="text-gray-500 dark:text-gray-400">
          Carregando previsão...
        </p>
      </div>
    );
  }

  // Exibir previsão para os próximos 5 dias
  const dailyForecast = forecast.filter((item, index) => index % 8 === 0).slice(0, 5);

  return (
    <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-4">
      {dailyForecast.map((item, index) => (
        <div 
          key={index} 
          className="bg-white/5 dark:bg-white/5 rounded-xl p-4 text-center"
        >
          <p className="text-sm text-gray-500 dark:text-gray-400 mb-2">
            {format(new Date(item.date), 'EEEE', { locale: ptBR })}
          </p>
          
          <div className="flex justify-center mb-2">
            <img 
              src={`/images/weather-icons/${item.weather_icon}.svg`} 
              alt={item.weather_condition}
              className="w-12 h-12"
            />
          </div>
          
          <p className="font-medium text-lg">{Math.round(item.temperature)}°C</p>
          
          <p className="text-xs text-gray-500 dark:text-gray-400 mt-1">
            {item.weather_condition}
          </p>
        </div>
      ))}
    </div>
  );
};

export default WeatherForecast;
```

Criar arquivo `components/weather/CityComparison.tsx`:
```tsx
'use client';

import { useEffect } from 'react';
import { useWeatherStore } from '@/app/store/weatherStore';

const CityComparison: React.FC = () => {
  const { cityComparison, setCityComparison, fetchWeatherData } = useWeatherStore();

  // Cidades para comparação
  const citiesToCompare = [
    'Washington D.C.',
    'Oklahoma City',
    'Philadelphia',
    'San Francisco',
    'New York City',
    'South Dakota',
    'North Dakota'
  ];

  useEffect(() => {
    // Simular busca de dados para comparação
    // Em uma implementação real, você buscaria dados reais da API
    const mockData = citiesToCompare.map((city, index) => {
      const temperatures = [20, 17, 14, 12, 10, -8, -9];
      const colors = [
        '#FFB347', // laranja
        '#FF6B6B', // vermelho
        '#4ECDC4', // turquesa
        '#7FB3D5', // azul
        '#9B59B6', // roxo
        '#3498DB', // azul escuro
        '#2C3E50'  // azul muito escuro
      ];
      
      return {
        city,
        temperature: temperatures[index],
        color: colors[index]
      };
    });
    
    setCityComparison(mockData);
  }, []);

  if (!cityComparison || cityComparison.length === 0) {
    return (
      <div className="flex items-center justify-center h-40">
        <p className="text-gray-500 dark:text-gray-400">
          Carregando comparação...
        </p>
      </div>
    );
  }

  return (
    <div className="space-y-4">
      <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
        {cityComparison.map((item, index) => (
          <div key={index} className="text-center">
            <div className="text-2xl font-light mb-1">
              {item.temperature}°
            </div>
            <div className="text-sm text-gray-600 dark:text-gray-300">
              {item.city}
            </div>
            <div 
              className="h-1 mt-2 rounded-full" 
              style={{ backgroundColor: item.color }}
            ></div>
          </div>
        ))}
      </div>
    </div>
  );
};

export default CityComparison;
```

Criar arquivo `components/theme/ThemeToggle.tsx`:
```tsx
'use client';

import { useTheme } from '@/app/context/ThemeContext';
import { FiSun, FiMoon } from 'react-icons/fi';
import Toggle from '../ui/Toggle';

const ThemeToggle: React.FC = () => {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <div className="flex items-center space-x-2">
      <FiSun className="text-yellow-500" />
      <Toggle
        enabled={theme === 'dark'}
        onChange={toggleTheme}
      />
      <FiMoon className="text-gray-600 dark:text-gray-300" />
    </div>
  );
};

export default ThemeToggle;
```

## 6. Utilitários e Helpers

### 6.1. Funções Utilitárias
Criar arquivo `lib/utils.ts`:
```typescript
import { format, parseISO } from 'date-fns';
import { ptBR } from 'date-fns/locale';
import { 
  OpenWeatherMapResponse, 
  OpenWeatherMapForecastResponse,
  OpenWeatherMapAirPollutionResponse,
  OpenWeatherMapAlertsResponse
} from '../types/api';
import { 
  WeatherData, 
  ForecastData, 
  WeatherAlert, 
  AirQualityData 
} from '../types/weather';

// Converter resposta da API para o formato interno de dados do clima
export const mapCurrentWeather = (data: OpenWeatherMapResponse): WeatherData => {
  const now = new Date();
  
  return {
    city: data.name,
    country: data.sys.country,
    temperature: data.main.temp,
    feels_like: data.main.feels_like,
    humidity: data.main.humidity,
    wind_speed: data.wind.speed,
    wind_direction: getWindDirection(data.wind.deg),
    weather_condition: data.weather[0].main,
    weather_description: data.weather[0].description,
    weather_icon: data.weather[0].icon,
    precipitation: data.rain ? (data.rain['1h'] || 0) * 100 : 0,
    date: format(now, 'yyyy-MM-dd'),
    time: format(now, 'HH:mm:ss'),
  };
};

// Converter resposta da API para o formato interno de previsão
export const mapForecastData = (data: OpenWeatherMapForecastResponse): ForecastData[] => {
  return data.list.map(item => {
    const date = parseISO(item.dt_txt);
    
    return {
      date: format(date, 'yyyy-MM-dd'),
      time: format(date, 'HH:mm:ss'),
      temperature: item.main.temp,
      feels_like: item.main.feels_like,
      humidity: item.main.humidity,
      wind_speed: item.wind.speed,
      weather_condition: item.weather[0].main,
      weather_icon: item.weather[0].icon,
      precipitation: item.pop * 100,
    };
  });
};

// Converter resposta da API para o formato interno de qualidade do ar
export const mapAirQualityData = (data: OpenWeatherMapAirPollutionResponse): AirQualityData => {
  const aqi = data.list[0].main.aqi;
  const components = data.list[0].components;
  
  const descriptions = [
    'Excelente',
    'Boa',
    'Moderada',
    'Ruim',
    'Muito Ruim'
  ];
  
  return {
    index: aqi,
    description: descriptions[aqi - 1],
    components: {
      co: components.co,
      no2: components.no2,
      o3: components.o3,
      pm2_5: components.pm2_5,
      pm10: components.pm10,
      so2: components.so2,
    }
  };
};

// Converter resposta da API para o formato interno de alertas
export const mapWeatherAlerts = (data: OpenWeatherMapAlertsResponse): WeatherAlert[] => {
  if (!data.alerts || data.alerts.length === 0) {
    return [];
  }
  
  return data.alerts.map((alert, index) => {
    // Determinar severidade com base nas tags ou tipo de evento
    let severity: 'low' | 'medium' | 'high' = 'medium';
    
    const severeEvents = ['tornado', 'hurricane', 'flood', 'extreme'];
    const moderateEvents = ['storm', 'rain', 'snow', 'wind', 'fog'];
    
    const eventLower = alert.event.toLowerCase();
    
    if (severeEvents.some(term => eventLower.includes(term))) {
      severity = 'high';
    } else if (moderateEvents.some(term => eventLower.includes(term))) {
      severity = 'medium';
    } else {
      severity = 'low';
    }
    
    return {
      id: `alert-${index}`,
      type: alert.event,
      severity,
      title: alert.event,
      description: alert.description,
      start: new Date(alert.start * 1000).toISOString(),
      end: new Date(alert.end * 1000).toISOString(),
    };
  });
};

// Obter direção do vento com base nos graus
export const getWindDirection = (degrees: number): string => {
  const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
  const index = Math.round(degrees / 45) % 8;
  return directions[index];
};

// Formatar data para exibição
export const formatDate = (dateString: string, formatStr: string): string => {
  const date = parseISO(dateString);
  return format(date, formatStr, { locale: ptBR });
};

// Obter ícone com base na condição climática
export const getWeatherIcon = (condition: string, icon: string): string => {
  // Mapear códigos de ícones da API para nomes de arquivos locais
  const iconMap: Record<string, string> = {
    '01d': 'clear-day',
    '01n': 'clear-night',
    '02d': 'partly-cloudy-day',
    '02n': 'partly-cloudy-night',
    '03d': 'cloudy',
    '03n': 'cloudy',
    '04d': 'cloudy',
    '04n': 'cloudy',
    '09d': 'rain',
    '09n': 'rain',
    '10d': 'rain',
    '10n': 'rain',
    '11d': 'thunderstorm',
    '11n': 'thunderstorm',
    '13d': 'snow',
    '13n': 'snow',
    '50d': 'fog',
    '50n': 'fog',
  };
  
  return iconMap[icon] || 'cloudy';
};
```

### 6.2. Hooks Personalizados
Criar arquivo `hooks/useWeather.ts`:
```typescript
import { useState, useEffect } from 'react';
import { useWeatherStore } from '../store/weatherStore';

export const useWeather = (city: string) => {
  const { 
    fetchWeatherData, 
    currentWeather, 
    forecast, 
    alerts, 
    airQuality, 
    isLoading, 
    error 
  } = useWeatherStore();
  
  useEffect(() => {
    fetchWeatherData(city);
  }, [city, fetchWeatherData]);
  
  return {
    currentWeather,
    forecast,
    alerts,
    airQuality,
    isLoading,
    error
  };
};
```

Criar arquivo `hooks/useLocalStorage.ts`:
```typescript
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  // Estado para armazenar o valor
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }
    
    try {
      // Obter do localStorage pelo key
      const item = window.localStorage.getItem(key);
      // Analisar JSON armazenado ou retornar initialValue
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });
  
  // Função para atualizar o valor no localStorage
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      // Permitir que o valor seja uma função para ter a mesma API que useState
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      // Salvar estado
      setStoredValue(valueToStore);
      // Salvar no localStorage
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.log(error);
    }
  };
  
  return [storedValue, setValue] as const;
}
```

## 7. Testes e Otimização

### 7.1. Configuração de Testes
Instalar dependências de teste:
```bash
npm install -D jest @testing-library/react @testing-library/jest-dom jest-environment-jsdom
```

Criar arquivo `jest.config.js`:
```javascript
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    '^@/app/(.*)$': '<rootDir>/app/$1',
  },
  testEnvironment: 'jest-environment-jsdom',
};

module.exports = createJestConfig(customJestConfig);
```

Criar arquivo `jest.setup.js`:
```javascript
import '@testing-library/jest-dom/extend-expect';
```

### 7.2. Testes de Componentes
Criar arquivo `__tests__/components/CurrentWeather.test.tsx`:
```tsx
import { render, screen } from '@testing-library/react';
import CurrentWeather from '@/app/components/weather/CurrentWeather';
import { useWeatherStore } from '@/app/store/weatherStore';

// Mock do zustand store
jest.mock('@/app/store/weatherStore');

describe('CurrentWeather', () => {
  it('exibe mensagem de carregamento quando não há dados', () => {
    (useWeatherStore as jest.Mock).mockReturnValue({
      currentWeather: null
    });
    
    render(<CurrentWeather />);
    expect(screen.getByText('Carregando dados do clima...')).toBeInTheDocument();
  });
  
  it('exibe dados do clima quando disponíveis', () => {
    const mockWeather = {
      city: 'São Paulo',
      country: 'BR',
      temperature: 25,
      feels_like: 27,
      humidity: 80,
      wind_speed: 10,
      wind_direction: 'NE',
      weather_condition: 'Clouds',
      weather_description: 'Nublado',
      weather_icon: '04d',
      precipitation: 20,
      date: '2023-01-01',
      time: '12:00:00'
    };
    
    (useWeatherStore as jest.Mock).mockReturnValue({
      currentWeather: mockWeather
    });
    
    render(<CurrentWeather />);
    expect(screen.getByText('Clouds')).toBeInTheDocument();
    expect(screen.getByText(/with Nublado/i)).toBeInTheDocument();
  });
});
```

### 7.3. Otimização de Imagens
Configurar o Next.js para otimização de imagens no arquivo `next.config.js`:
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    domains: ['openweathermap.org'],
    formats: ['image/avif', 'image/webp'],
  },
  experimental: {
    optimizeCss: true,
  },
};

module.exports = nextConfig;
```

## 8. Implantação e Documentação

### 8.1. Preparação para Implantação
Atualizar scripts no `package.json`:
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

### 8.2. Documentação do Projeto
Criar arquivo `README.md`:
```markdown
# Weather Dashboard

Um dashboard de clima moderno e responsivo construído com Next.js e Tailwind CSS.

## Funcionalidades

- Exibição de dados meteorológicos em tempo real
- Previsão do tempo para os próximos dias
- Gráficos de temperatura
- Alertas meteorológicos
- Qualidade do ar
- Comparação entre cidades
- Busca por cidades
- Tema claro/escuro

## Tecnologias Utilizadas

- Next.js 13+ (App Router)
- TypeScript
- Tailwind CSS
- Chart.js
- Zustand (Gerenciamento de Estado)
- OpenWeatherMap API
- Framer Motion (Animações)
- Jest e Testing Library (Testes)

## Pré-requisitos

- Node.js 18.0.0 ou superior
- NPM ou Yarn

## Instalação

1. Clone o repositório:
```bash
git clone https://github.com/seu-usuario/weather-dashboard.git
cd weather-dashboard
```

2. Instale as dependências:
```bash
npm install
# ou
yarn install
```

3. Crie um arquivo `.env.local` na raiz do projeto e adicione sua chave API do OpenWeatherMap:
```
OPENWEATHERMAP_API_KEY=sua_chave_api_aqui
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

4. Inicie o servidor de desenvolvimento:
```bash
npm run dev
# ou
yarn dev
```

5. Abra [http://localhost:3000](http://localhost:3000) no seu navegador para ver o resultado.

## Estrutura do Projeto

- `app/` - Componentes, páginas e lógica da aplicação
  - `api/` - Rotas de API do Next.js
  - `components/` - Componentes reutilizáveis
  - `context/` - Contextos React
  - `hooks/` - Hooks personalizados
  - `lib/` - Funções utilitárias
  - `store/` - Gerenciamento de estado com Zustand
  - `types/` - Definições de tipos TypeScript
- `public/` - Arquivos estáticos
- `__tests__/` - Testes

## Implantação

Para construir a aplicação para produção:

```bash
npm run build
# ou
yarn build
```

Para iniciar a aplicação em produção:

```bash
npm run start
# ou
yarn start
```

## Licença

MIT
```

## 9. Cronograma de Implementação

### Fase 1: Configuração e Estrutura Básica (2-3 dias)
- Criar projeto Next.js
- Configurar Tailwind CSS
- Definir estrutura de pastas
- Configurar variáveis de ambiente
- Implementar tipos e interfaces

### Fase 2: Integração com API (2-3 dias)
- Implementar serviços de API
- Criar rotas de API no Next.js
- Implementar transformadores de dados
- Configurar gerenciamento de estado

### Fase 3: Componentes de UI (3-4 dias)
- Implementar componentes básicos de UI
- Criar componentes de layout
- Implementar componentes específicos de clima
- Adicionar tema claro/escuro

### Fase 4: Funcionalidades Interativas (2-3 dias)
- Implementar busca por cidades
- Adicionar histórico de pesquisas
- Implementar alertas de clima
- Criar gráficos de temperatura

### Fase 5: Testes e Otimização (2-3 dias)
- Escrever testes unitários
- Otimizar desempenho
- Melhorar acessibilidade
- Garantir responsividade

### Fase 6: Finalização e Implantação (1-2 dias)
- Documentar o projeto
- Preparar para implantação
- Implantar em ambiente de produção
- Realizar testes finais

## 10. Considerações Finais

Este roadmap fornece um guia detalhado para implementar um dashboard de clima moderno e responsivo usando Next.js e Tailwind CSS. A estrutura proposta segue as melhores práticas de desenvolvimento front-end, com foco em:

1. **Organização de código** - Estrutura de pastas clara e modular
2. **Tipagem forte** - Uso de TypeScript para maior segurança
3. **Componentização** - Componentes reutilizáveis e bem definidos
4. **Gerenciamento de estado** - Uso de Zustand para estado global
5. **Responsividade** - Design adaptável a diferentes dispositivos
6. **Temas** - Suporte a modo claro/escuro
7. **Performance** - Otimizações para carregamento rápido
8. **Testabilidade** - Estrutura preparada para testes

Seguindo este roadmap, você terá um dashboard de clima completo e profissional, similar ao mostrado na imagem de referência, com todas as funcionalidades necessárias para uma experiência de usuário de alta qualidade.

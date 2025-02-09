import React, { useState, useEffect } from 'react';
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts';
import { getDatabase, ref, onValue } from "firebase/database";
import { initializeApp } from "firebase/app";
import { Loader } from "lucide-react";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  databaseURL: "YOUR_DATABASE_URL",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

export default function ElectricityMonitor() {
  const [data, setData] = useState([]);
  const [currentUsage, setCurrentUsage] = useState(0);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const meterRef = ref(db, 'meterData');
    onValue(meterRef, (snapshot) => {
      const meterData = snapshot.val();
      if (meterData) {
        const formattedData = Object.keys(meterData).map((key) => ({
          time: key,
          usage: meterData[key].usage,
        }));
        setData(formattedData);
        setCurrentUsage(meterData[Object.keys(meterData).pop()].usage);
      }
      setLoading(false);
    });
  }, []);

  return (
    <div className="p-6 flex flex-col items-center bg-gray-100 min-h-screen">
      <h1 className="text-3xl font-bold mb-6 text-blue-600">Real-Time Electricity Monitoring</h1>
      <Card className="w-full max-w-md p-6 mb-6 shadow-lg border border-gray-200 bg-white">
        <CardContent className="text-center">
          <h2 className="text-2xl font-semibold text-gray-700">Current Usage</h2>
          {loading ? (
            <Loader className="animate-spin mx-auto mt-4" />
          ) : (
            <p className="text-5xl font-bold text-green-600 mt-4">{currentUsage} kWh</p>
          )}
        </CardContent>
      </Card>
      <div className="w-full max-w-3xl h-96 p-4 bg-white shadow-lg rounded-lg">
        <ResponsiveContainer width="100%" height="100%">
          <LineChart data={data}>
            <CartesianGrid strokeDasharray="3 3" />
            <XAxis dataKey="time" />
            <YAxis />
            <Tooltip />
            <Line type="monotone" dataKey="usage" stroke="#1E40AF" strokeWidth={2} />
          </LineChart>
        </ResponsiveContainer>
      </div>
      <Button className="mt-6 bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg shadow-md">
        Refresh Data
      </Button>
    </div>
  );
}

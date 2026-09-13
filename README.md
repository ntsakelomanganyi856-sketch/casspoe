import React, { useState } from 'react';
import {
  StyleSheet,
  Text,
  View,
  TextInput,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
} from 'react-native';
import { StatusBar } from 'expo-status-bar';

export default function App() {
  const [ice, setIce] = useState<string>('');
  const [part1, setPart1] = useState<string>('');
  const [part2, setPart2] = useState<string>('');
  const [finalMark, setFinalMark] = useState<string>('');

  const [cass, setCass] = useState<number | null>(null);
  const [poeNeeded, setPoeNeeded] = useState<number | null>(null);
  const [error, setError] = useState<string>('');

  const calculate = () => {
    setError('');
    const iceVal = parseFloat(ice);
    const p1Val = parseFloat(part1);
    const p2Val = parseFloat(part2);
    const fmVal = parseFloat(finalMark);

    if (
      isNaN(iceVal) ||
      isNaN(p1Val) ||
      isNaN(p2Val) ||
      isNaN(fmVal)
    ) {
      setError('Please fill in all fields with valid numbers (0-100)');
      return;
    }

    if ([iceVal, p1Val, p2Val, fmVal].some(v => v < 0 || v > 100)) {
      setError('Marks must be between 0 and 100');
      return;
    }

    // CASS Calculation: ICE is already out of 10%, P1*0.25, P2*0.3
    // Assuming user enters marks out of 100 for each component, we convert:
    // ICE contribution = ICE * 0.10
    // But from your formula: CASS = ICE + P1*0.25 + P2*0.3
    // This implies ICE input is already weighted (e.g 10% max).
    // I will use the formula exactly as given, but also support 0-100 input.
    
    // Formula as per screenshot: CASS = ICE + P1*0.25 + P2*0.3
    // If ICE entered as /100, we do ICE * 0.10
    const calculatedCass = (iceVal * 0.10) + (p1Val * 0.25) + (p2Val * 0.30);
    
    // POE_Needed = (FM - CASS) / 0.35
    const calculatedPoeNeeded = (fmVal - calculatedCass) / 0.35;

    setCass(calculatedCass);
    setPoeNeeded(calculatedPoeNeeded);
  };

  const reset = () => {
    setIce('');
    setPart1('');
    setPart2('');
    setFinalMark('');
    setCass(null);
    setPoeNeeded(null);
    setError('');
  };

  const getPoeStatus = () => {
    if (poeNeeded === null) return null;
    if (poeNeeded > 100) return { text: 'Impossible! You need >100% in POE', color: '#D32F2F' };
    if (poeNeeded < 0) return { text: 'You already achieved your target!', color: '#2E7D32' };
    if (poeNeeded > 80) return { text: 'Very tough, but possible', color: '#EF6C00' };
    return { text: 'Achievable!', color: '#2E7D32' };
  };

  const status = getPoeStatus();

  return (
    <KeyboardAvoidingView style={styles.container} behavior={Platform.OS === 'ios' ? 'padding' : 'height'}>
      <ScrollView contentContainerStyle={styles.scrollContent} keyboardShouldPersistTaps="handled">
        <StatusBar style="light" />
        
        <View style={styles.header}>
          <Text style={styles.title}>CASS & POE Calculator</Text>
          <Text style={styles.subtitle}>ICE 10% | Part 1 25% | Part 2 30% | POE 35%</Text>
        </View>

        <View style={styles.card}>
          <InputField label="ICE Mark (/100)" value={ice} onChange={setIce} placeholder="e.g. 85" />
          <InputField label="Part 1 Mark (/100)" value={part1} onChange={setPart1} placeholder="e.g. 70" />
          <InputField label="Part 2 Mark (/100)" value={part2} onChange={setPart2} placeholder="e.g. 65" />
          <InputField label="Expected Final Mark (/100)" value={finalMark} onChange={setFinalMark} placeholder="e.g. 75" isLast />

          {error ? <Text style={styles.errorText}>{error}</Text> : null}

          <View style={styles.buttonRow}>
            <TouchableOpacity style={styles.calculateBtn} onPress={calculate}>
              <Text style={styles.btnText}>Calculate</Text>
            </TouchableOpacity>
            <TouchableOpacity style={styles.resetBtn} onPress={reset}>
              <Text style={styles.resetBtnText}>Reset</Text>
            </TouchableOpacity>
          </View>
        </View>

        {cass !== null && poeNeeded !== null && (
          <View style={styles.resultCard}>
            <Text style={styles.resultTitle}>Results</Text>
            
            <View style={styles.resultRow}>
              <Text style={styles.resultLabel}>CASS (65% total):</Text>
              <Text style={styles.resultValue}>{cass.toFixed(2)}%</Text>
            </View>

            <View style={styles.divider} />

            <View style={styles.resultRow}>
              <Text style={styles.resultLabel}>POE Needed:</Text>
              <Text style={[styles.resultValue, { color: status?.color }]}>{poeNeeded.toFixed(2)}%</Text>
            </View>
            
            {status && <Text style={[styles.statusText, { color: status.color }]}>{status.text}</Text>}

            <View style={styles.formulaBox}>
              <Text style={styles.formulaText}>Formula used:</Text>
              <Text style={styles.formulaText}>CASS = ICE*0.10 + P1*0.25 + P2*0.30</Text>
              <Text style={styles.formulaText}>POE_Needed = (FM - CASS) / 0.35</Text>
            </View>
          </View>
        )}
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

type InputProps = {
  label: string;
  value: string;
  onChange: (t: string) => void;
  placeholder: string;
  isLast?: boolean;
};

const InputField = ({ label, value, onChange, placeholder, isLast }: InputProps) => (
  <View style={[styles.inputGroup, isLast && { marginBottom: 0 }]}>
    <Text style={styles.label}>{label}</Text>
    <TextInput
      style={styles.input}
      value={value}
      onChangeText={onChange}
      placeholder={placeholder}
      placeholderTextColor="#999"
      keyboardType="numeric"
    />
  </View>
);

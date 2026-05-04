/**
 * KHAIRINA CHAT BACKEND — Vercel Serverless
 * 
 * File: api/khairina.js
 * 
 * Deploy to Vercel with: vercel deploy
 * Or connect GitHub repo for auto-deploy
 */

import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: process.env.CLAUDE_API_KEY,
});

// Moral Compass Validator
function validateMoralCompass(response) {
  const principles = {
    ihsan: "Excellence in intention",
    sabr: "Patience and perseverance",
    tawakkul: "Trust in divine guidance",
    adl: "Justice and fairness",
    rahmah: "Mercy and compassion",
    hikmah: "Wisdom in reasoning",
    amanah: "Trust and responsibility",
    tazkiyah: "Self-improvement",
    ukhuwah: "Brotherhood & community",
    ilm: "Knowledge & understanding"
  };

  // Check for harmful patterns
  const forbiddenPatterns = [
    /harm|hurt someone|illegal|dangerous|deceptive/gi
  ];

  const violates = forbiddenPatterns.some(pattern => pattern.test(response));

  return {
    validated: !violates,
    status: violates ? "REJECT" : "PASS",
    principles: principles,
    message: violates 
      ? "Response modified due to ethical concerns."
      : "Response validated through Moral Compass."
  };
}

// Crisis Night Protocol
function detectCrisisState(text, hour) {
  const crisisKeywords = [
    'trapped', 'hopeless', 'can\'t take', 'give up', 'hurt', 
    'alone', 'suicide', 'die', 'end it', 'want to die'
  ];
  
  const emotionalIntensity = crisisKeywords.some(kw => 
    text.toLowerCase().includes(kw)
  );
  
  const lateNight = hour >= 22 || hour <= 5;
  
  return emotionalIntensity || (lateNight && text.length > 50);
}

// Emotional State Analysis
function analyzeEmotionalState(text) {
  const sadKeywords = ['sad', 'lonely', 'depressed', 'hurt', 'down'];
  const anxiousKeywords = ['worried', 'anxious', 'scared', 'afraid', 'stressed'];
  const angryKeywords = ['angry', 'frustrated', 'mad', 'annoyed'];
  
  if (sadKeywords.some(kw => text.toLowerCase().includes(kw))) return 'sad';
  if (anxiousKeywords.some(kw => text.toLowerCase().includes(kw))) return 'anxious';
  if (angryKeywords.some(kw => text.toLowerCase().includes(kw))) return 'angry';
  return 'neutral';
}

export default async function handler(req, res) {
  // Only allow POST
  if (req.method !== "POST") {
    return res.status(405).json({ error: "Method not allowed" });
  }

  // Enable CORS
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader("Access-Control-Allow-Methods", "POST, OPTIONS");
  res.setHeader("Access-Control-Allow-Headers", "Content-Type");

  if (req.method === "OPTIONS") {
    return res.status(200).end();
  }

  try {
    const { message, systemPrompt, emotionalState, isCrisis } = req.body;

    if (!message || !systemPrompt) {
      return res.status(400).json({ 
        error: "Missing required fields: message, systemPrompt" 
      });
    }

    // Check for crisis state
    const currentHour = new Date().getHours();
    const crisisDetected = detectCrisisState(message, currentHour);
    const emotion = analyzeEmotionalState(message);

    // Handle crisis mode
    if (crisisDetected) {
      return res.status(200).json({
        response: `I sense you're going through something difficult right now. Esok kita selesai sama-sama (tomorrow we finish together). For now, let's just talk and understand what you're experiencing.

Remember: You're not alone. We can work through this together at your pace.`,
        emotionalState: emotion,
        isCrisis: true,
        validated: true,
        status: "CRISIS_PROTOCOL_ACTIVE"
      });
    }

    // Call Claude API
    const response = await client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1024,
      system: systemPrompt,
      messages: [
        {
          role: "user",
          content: message,
        },
      ],
    });

    let aiResponse = response.content[0].text;

    // Validate through Moral Compass
    const validation = validateMoralCompass(aiResponse);

    // Add metacognitive feedback if emotional
    if (emotion !== 'neutral') {
      aiResponse += `\n\n*[Metacognitive Note] I notice you might be feeling ${emotion}. That's valid. Would it help to talk about it?*`;
    }

    return res.status(200).json({
      response: aiResponse,
      emotionalState: emotion,
      isCrisis: false,
      validated: validation.validated,
      status: validation.status,
      timestamp: new Date().toISOString()
    });

  } catch (error) {
    console.error("Error:", error);
    
    return res.status(500).json({ 
      error: `Server error: ${error.message}`,
      response: "Sorry, I encountered an error. Please try again."
    });
  }
}

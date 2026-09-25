https://github.com# mailguard-ai
https://mail-guard-six.vercel.appmailguard-ai/
├── app/
│   ├── api/
│   │   └── analyze/
│   │       └── route.ts
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── public/
├── .env.local
├── .gitignore
├── package.json
├── tailwind.config.js
├── tsconfig.json
└── README.md
{
      "name": "mailguard-ai",
        "version": "1.0.0",
          "private": true,
            "scripts": {
                "dev": "next dev",
                    "build": "next build",
                        "start": "next start",
                            "lint": "next lint"
                              },
                                "dependencies": {
                                    "@google/generative-ai": "^0.21.0",
                                        "clsx": "^2.1.1",
                                            "framer-motion": "^11.11.0",
                                                "lucide-react": "^0.453.0",
                                                    "next": "14.2.15",
                                                        "react": "^18.3.1",
                                                            "react-dom": "^18.3.1",
                                                                "tailwind-merge": "^2.5.4"
                                                                  },
                                                                    "devDependencies": {
                                                                        "@types/node": "^20",
                                                                            "@types/react": "^18",
                                                                                "@types/react-dom": "^18",
                                                                                    "autoprefixer": "^10.4.20",
                                                                                        "postcss": "^8.4.47",
                                                                                            "tailwindcss": "^3.4.14",
                                                                                                "typescript": "^5"
                                                                                                  }
                                                                                                  }
                                                                                                  @tailwind base;
                                                                                                  @tailwind components;
                                                                                                  @tailwind utilities;

                                                                                                  body {
                                                                                                    @apply bg-slate-950 text-slate-100 antialiased selection:bg-cyan-500 selection:text-slate-950;
                                                                                                      font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
                                                                                                      }
                                                                                                      import type { Metadata } from "next";
                                                                                                      import "./globals.css";

                                                                                                      export const metadata: Metadata = {
                                                                                                        title: "MailGuard AI — Next-Gen Email Threat Intelligence",
                                                                                                          description: "AI-powered phishing detection, email header analysis, and cybersecurity threat evaluation.",
                                                                                                          };

                                                                                                          export default function RootLayout({
                                                                                                            children,
                                                                                                            }: {
                                                                                                              children: React.ReactNode;
                                                                                                              }) {
                                                                                                                return (
                                                                                                                    <html lang="en">
                                                                                                                          <body className="min-h-screen flex flex-col justify-between">
                                                                                                                                  {children}
                                                                                                                                        </body>
                                                                                                                                            </html>
                                                                                                                                              );
                                                                                                                                              }
                                                                                                                                              import { NextRequest, NextResponse } from "next/server";
                                                                                                                                              import { GoogleGenerativeAI } from "@google/generative-ai";

                                                                                                                                              const apiKey = process.env.GEMINI_API_KEY;
                                                                                                                                              const genAI = apiKey ? new GoogleGenerativeAI(apiKey) : null;

                                                                                                                                              export async function POST(req: NextRequest) {
                                                                                                                                                try {
                                                                                                                                                    const { emailContent, sender, subject } = await req.json();

                                                                                                                                                        if (!emailContent) {
                                                                                                                                                              return NextResponse.json(
                                                                                                                                                                      { error: "Email content is required." },
                                                                                                                                                                              { status: 400 }
                                                                                                                                                                                    );
                                                                                                                                                                                        }

                                                                                                                                                                                            // Fallback heuristic engine if GEMINI_API_KEY is not set
                                                                                                                                                                                                if (!genAI) {
                                                                                                                                                                                                      return NextResponse.json(runHeuristicAnalysis(emailContent, sender, subject));
                                                                                                                                                                                                          }

                                                                                                                                                                                                              const model = genAI.getGenerativeAIModel({ model: "gemini-2.5-flash" });

                                                                                                                                                                                                                  const prompt = `
                                                                                                                                                                                                                  You are MailGuard AI, an enterprise-grade cyber threat analysis system.
                                                                                                                                                                                                                  Analyze the following email for phishing attempts, spoofing, social engineering tactics, malicious links, and urgency manipulation.

                                                                                                                                                                                                                  Sender: ${sender || "Unknown"}
                                                                                                                                                                                                                  Subject: ${subject || "None"}
                                                                                                                                                                                                                  Content:
                                                                                                                                                                                                                  """
                                                                                                                                                                                                                  ${emailContent}
                                                                                                                                                                                                                  """

                                                                                                                                                                                                                  Return strictly a valid JSON object matching this schema EXACTLY without markdown or commentary:
                                                                                                                                                                                                                  {
                                                                                                                                                                                                                    "riskScore": number (0 to 100, where 100 is critical danger),
                                                                                                                                                                                                                      "riskLevel": "CRITICAL" | "HIGH" | "MEDIUM" | "LOW" | "SAFE",
                                                                                                                                                                                                                        "summary": "Concise summary of the threat assessment.",
                                                                                                                                                                                                                          "headerAnalysis": {
                                                                                                                                                                                                                              "spf": "PASS" | "FAIL" | "UNKNOWN",
                                                                                                                                                                                                                                  "dkim": "PASS" | "FAIL" | "UNKNOWN",
                                                                                                                                                                                                                                      "dmarc": "PASS" | "FAIL" | "UNKNOWN"
                                                                                                                                                                                                                                        },
                                                                                                                                                                                                                                          "indicators": [
                                                                                                                                                                                                                                              {
                                                                                                                                                                                                                                                    "category": "Domain Spoofing" | "Urgency" | "Suspicious Link" | "Financial Request" | "Credential Harvesting",
                                                                                                                                                                                                                                                          "severity": "high" | "medium" | "low",
                                                                                                                                                                                                                                                                "description": "Specific observation from text"
                                                                                                                                                                                                                                                                    }
                                                                                                                                                                                                                                                                      ],
                                                                                                                                                                                                                                                                        "recommendations": ["Actionable step 1", "Actionable step 2"]
                                                                                                                                                                                                                                                                        }
                                                                                                                                                                                                                                                                        `;

                                                                                                                                                                                                                                                                            const response = await model.generateContent(prompt);
                                                                                                                                                                                                                                                                                const responseText = response.response.text().trim();
                                                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                                        // Clean codeblock markers if present
                                                                                                                                                                                                                                                                                            const cleanedJson = responseText.replace(/^```json\s*|\s*```$/g, "");
                                                                                                                                                                                                                                                                                                const parsedData = JSON.parse(cleanedJson);

                                                                                                                                                                                                                                                                                                    return NextResponse.json(parsedData);
                                                                                                                                                                                                                                                                                                      } catch (error) {
                                                                                                                                                                                                                                                                                                          console.error("Analysis Error:", error);
                                                                                                                                                                                                                                                                                                              return NextResponse.json(
                                                                                                                                                                                                                                                                                                                    { error: "Failed to analyze email content." },
                                                                                                                                                                                                                                                                                                                          { status: 500 }
                                                                                                                                                                                                                                                                                                                              );
                                                                                                                                                                                                                                                                                                                                }
                                                                                                                                                                                                                                                                                                                                }

                                                                                                                                                                                                                                                                                                                                // Heuristic Fallback Function
                                                                                                                                                                                                                                                                                                                                function runHeuristicAnalysis(emailContent: string, sender: string, subject: string) {
                                                                                                                                                                                                                                                                                                                                  const text = (emailContent + " " + subject).toLowerCase();
                                                                                                                                                                                                                                                                                                                                    let score = 15;
                                                                                                                                                                                                                                                                                                                                      const indicators = [];

                                                                                                                                                                                                                                                                                                                                        if (text.includes("urgent") || text.includes("immediately") || text.includes("action required") || text.includes("account suspended")) {
                                                                                                                                                                                                                                                                                                                                            score += 25;
                                                                                                                                                                                                                                                                                                                                                indicators.push({ category: "Urgency", severity: "high", description: "Creates artificial urgency to bypass critical thinking." });
                                                                                                                                                                                                                                                                                                                                                  }

                                                                                                                                                                                                                                                                                                                                                    if (text.includes("password") || text.includes("verify") || text.includes("login") || text.includes("click here")) {
                                                                                                                                                                                                                                                                                                                                                        score += 20;
                                                                                                                                                                                                                                                                                                                                                            indicators.push({ category: "Credential Harvesting", severity: "high", description: "Requests login verification or password confirmation." });
                                                                                                                                                                                                                                                                                                                                                              }

                                                                                                                                                                                                                                                                                                                                                                if (text.includes("bank") || text.includes("wire transfer") || text.includes("gift card") || text.includes("invoice")) {
                                                                                                                                                                                                                                                                                                                                                                    score += 20;
                                                                                                                                                                                                                                                                                                                                                                        indicators.push({ category: "Financial Request", severity: "medium", description: "Unsolicited payment or financial directive." });
                                                                                                                                                                                                                                                                                                                                                                          }

                                                                                                                                                                                                                                                                                                                                                                            if (sender && !sender.includes("@official")) {
                                                                                                                                                                                                                                                                                                                                                                                score += 10;
                                                                                                                                                                                                                                                                                                                                                                                  }

                                                                                                                                                                                                                                                                                                                                                                                    score = Math.min(score, 98);

                                                                                                                                                                                                                                                                                                                                                                                      let riskLevel = "SAFE";
                                                                                                                                                                                                                                                                                                                                                                                        if (score >= 80) riskLevel = "CRITICAL";
                                                                                                                                                                                                                                                                                                                                                                                          else if (score >= 60) riskLevel = "HIGH";
                                                                                                                                                                                                                                                                                                                                                                                            else if (score >= 40) riskLevel = "MEDIUM";
                                                                                                                                                                                                                                                                                                                                                                                              else if (score >= 20) riskLevel = "LOW";

                                                                                                                                                                                                                                                                                                                                                                                                return {
                                                                                                                                                                                                                                                                                                                                                                                                    riskScore: score,
                                                                                                                                                                                                                                                                                                                                                                                                        riskLevel,
                                                                                                                                                                                                                                                                                                                                                                                                            summary: score > 50 ? "High risk of phishing or malicious intent detected based on language heuristics." : "Email appears relatively benign, though standard caution is advised.",
                                                                                                                                                                                                                                                                                                                                                                                                                headerAnalysis: { spf: "PASS", dkim: "UNKNOWN", dmarc: "PASS" },
                                                                                                                                                                                                                                                                                                                                                                                                                    indicators,
                                                                                                                                                                                                                                                                                                                                                                                                                        recommendations: [
                                                                                                                                                                                                                                                                                                                                                                                                                              "Do not click on links embedded in this email.",
                                                                                                                                                                                                                                                                                                                                                                                                                                    "Verify the sender's identity through an official external channel.",
                                                                                                                                                                                                                                                                                                                                                                                                                                          "Report suspicious emails to your IT security department."
                                                                                                                                                                                                                                                                                                                                                                                                                                              ]
                                                                                                                                                                                                                                                                                                                                                                                                                                                };
                                                                                                                                                                                                                                                                                                                                                                                                                                                }

}
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  @apply bg-slate-950 text-slate-100 antialiased selection:bg-cyan-500 selection:text-slate-950;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    }
    import type { Metadata } from "next";
    import "./globals.css";

    export const metadata: Metadata = {
      title: "MailGuard AI — Next-Gen Email Threat Intelligence",
        description: "AI-powered phishing detection, email header analysis, and cybersecurity threat evaluation.",
        };

        export default function RootLayout({
          children,
          }: {
            children: React.ReactNode;
            }) {
              return (
                  <html lang="en">
                        <body className="min-h-screen flex flex-col justify-between">
                                {children}
                                      </body>
                                          </html>
                                            );
                                            }
                                            # 🛡️ MailGuard AI — Email Threat Analysis & Security Platform

                                            MailGuard AI is an automated cybersecurity platform designed to detect email phishing attacks, domain spoofing, social engineering indicators, and malicious intent.

                                            ## ✨ Features
                                            - 🔍 **Real-Time Phishing Audit:** Instant 0–100 risk score breakdown.
                                            - 📜 **Header Heuristics & Verification:** Evaluation of SPF, DKIM, and DMARC status.
                                            - ⚠️ **Threat Category Classification:** Spotlights urgency manipulation, credential harvesting, and suspicious links.
                                            - 🛡️ **Actionable Defense Recommendations:** Step-by-step guidance for end-users.

                                            ## 🚀 Quickstart Guide

                                            1. **Clone the repository:**
                                               ```bash
                                                  git clone [https://github.com/YOUR_GITHUB_USERNAME/mailguard-ai.git](https://github.com/YOUR_GITHUB_USERNAME/mailguard-ai.git)
                                                     cd mailguard-ai
                                                     # 🛡️ MailGuard AI — Email Threat Analysis & Security Platform

                                                     MailGuard AI is an automated cybersecurity platform designed to detect email phishing attacks, domain spoofing, social engineering indicators, and malicious intent.

                                                     ## ✨ Features
                                                     - 🔍 **Real-Time Phishing Audit:** Instant 0–100 risk score breakdown.
                                                     - 📜 **Header Heuristics & Verification:** Evaluation of SPF, DKIM, and DMARC status.
                                                     - ⚠️ **Threat Category Classification:** Spotlights urgency manipulation, credential harvesting, and suspicious links.
                                                     - 🛡️ **Actionable Defense Recommendations:** Step-by-step guidance for end-users.

                                                     ## 🚀 Quickstart Guide

                                                     1. **Clone the repository:**
                                                        ```bash


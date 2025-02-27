/** @jsxImportSource https://esm.sh/react@18.2.0 */
import React, { useState } from "https://esm.sh/react@18.2.0";
import { createRoot } from "https://esm.sh/react-dom@18.2.0/client";

function App() {
  const [articleText, setArticleText] = useState("");
  const [analysisResult, setAnalysisResult] = useState(null);
  const [isLoading, setIsLoading] = useState(false);

  const handleAnalyze = async (e) => {
    e.preventDefault();
    setIsLoading(true);
    setAnalysisResult(null);

    try {
      const response = await fetch("/analyze", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ text: articleText })
      });

      const result = await response.json();
      setAnalysisResult(result);
    } catch (error) {
      console.error("Analysis failed", error);
      setAnalysisResult({ 
        isFakeNews: true, 
        confidence: 0, 
        reason: "Error in analysis" 
      });
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div style={{ 
      maxWidth: "600px", 
      margin: "0 auto", 
      padding: "20px", 
      fontFamily: "Arial, sans-serif" 
    }}>
      <h1>🕵️ Fake News Detector</h1>
      <form onSubmit={handleAnalyze}>
        <textarea
          placeholder="Paste the news article text here..."
          value={articleText}
          onChange={(e) => setArticleText(e.target.value)}
          rows={10}
          style={{ 
            width: "100%", 
            marginBottom: "10px", 
            padding: "10px" 
          }}
          required
        />
        <button 
          type="submit" 
          disabled={isLoading}
          style={{ 
            width: "100%", 
            padding: "10px", 
            backgroundColor: isLoading ? "#cccccc" : "#4CAF50",
            color: "white",
            border: "none",
            cursor: isLoading ? "default" : "pointer"
          }}
        >
          {isLoading ? "Analyzing..." : "Detect Fake News"}
        </button>
      </form>

      {analysisResult && (
        <div 
          style={{ 
            marginTop: "20px", 
            padding: "15px", 
            backgroundColor: analysisResult.isFakeNews ? "#ffdddd" : "#ddffdd",
            borderRadius: "5px"
          }}
        >
          <h2>{analysisResult.isFakeNews ? "⚠️ Potential Fake News" : "✅ Likely Genuine News"}</h2>
          <p>Confidence: {(analysisResult.confidence * 100).toFixed(2)}%</p>
          <p>Reason: {analysisResult.reason}</p>
        </div>
      )}
      <p>
        <a 
          href={import.meta.url.replace("esm.town", "val.town")} 
          target="_top"
        >
          View Source
        </a>
      </p>
    </div>
  );
}

function client() {
  createRoot(document.getElementById("root")).render(<App />);
}
if (typeof document !== "undefined") { client(); }

export default async function server(request: Request): Promise<Response> {
  if (request.method === "POST") {
    const { text } = await request.json();
    
    // Simple fake news detection logic
    const result = detectFakeNews(text);
    
    return new Response(JSON.stringify(result), {
      headers: { "Content-Type": "application/json" }
    });
  }

  return new Response(`
    <html>
      <head>
        <title>Fake News Detector</title>
        <script src="https://esm.town/v/std/catch"></script>
      </head>
      <body>
        <div id="root"></div>
        <script type="module" src="${import.meta.url}"></script>
      </body>
    </html>
  `, {
    headers: { "Content-Type": "text/html" }
  });
}

function detectFakeNews(text: string) {
  // Very basic fake news detection algorithm
  const suspiciousKeywords = [
    "shocking", "unbelievable", "secret", "conspiracy", 
    "exclusive", "breaking", "viral", "exposed"
  ];

  const wordCount = text.toLowerCase().split(/\s+/).length;
  const suspiciousWordCount = suspiciousKeywords.filter(keyword => 
    text.toLowerCase().includes(keyword)
  ).length;

  const isFakeNews = (
    suspiciousWordCount > 2 || 
    wordCount < 50 || 
    text.includes("...") && suspiciousWordCount > 0
  );

  return {
    isFakeNews,
    confidence: isFakeNews ? 0.7 : 0.3,
    reason: isFakeNews 
      ? "Multiple suspicious keywords detected" 
      : "Text appears to be well-structured"
  };
}

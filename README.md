# AttendAI
Attendance monitoring system 
import { useState, useEffect, useRef } from "react";

// ============================================================
// DESIGN TOKENS
// Colors: Deep navy #0A0F1E, Electric indigo #4F46E5, 
//         Vivid cyan #06B6D4, Soft amber #F59E0B, 
//         Surface #111827, Text #F9FAFB
// Type: Display = "Space Grotesk", Body = "Inter"
// Signature: Real-time attendance pulse rings + live AI chat
// ============================================================

const COLORS = {
  bg: "#0A0F1E",
  surface: "#111827",
  surfaceHover: "#1F2937",
  border: "#1F2937",
  indigo: "#4F46E5",
  indigoLight: "#6366F1",
  cyan: "#06B6D4",
  amber: "#F59E0B",
  green: "#10B981",
  red: "#EF4444",
  text: "#F9FAFB",
  textMuted: "#6B7280",
  textSub: "#9CA3AF",
};

// ---- Simulated Data ----
const STUDENTS = [
  { id: 1, name: "Arjun Sharma", roll: "CS2021001", dept: "CSE", year: 3, avatar: "AS", risk: "low" },
  { id: 2, name: "Priya Nair", roll: "CS2021002", dept: "CSE", year: 3, avatar: "PN", risk: "medium" },
  { id: 3, name: "Rohan Mehta", roll: "CS2021003", dept: "CSE", year: 3, avatar: "RM", risk: "high" },
  { id: 4, name: "Sneha Iyer", roll: "CS2021004", dept: "CSE", year: 3, avatar: "SI", risk: "low" },
  { id: 5, name: "Karan Patel", roll: "CS2021005", dept: "CSE", year: 3, avatar: "KP", risk: "high" },
  { id: 6, name: "Meera Pillai", roll: "CS2021006", dept: "CSE", year: 3, avatar: "MP", risk: "medium" },
  { id: 7, name: "Dev Anand", roll: "CS2021007", dept: "CSE", year: 3, avatar: "DA", risk: "low" },
  { id: 8, name: "Ananya Roy", roll: "CS2021008", dept: "CSE", year: 3, avatar: "AR", risk: "medium" },
];

const SUBJECTS = ["Data Structures", "Machine Learning", "DBMS", "Networks", "OS"];

const generateAttendance = () => {
  const data = {};
  STUDENTS.forEach(s => {
    data[s.id] = {};
    SUBJECTS.forEach(sub => {
      const base = s.risk === "high" ? 55 : s.risk === "medium" ? 72 : 88;
      data[s.id][sub] = Math.min(100, Math.max(40, base + Math.floor(Math.random() * 16) - 8));
    });
  });
  return data;
};

const ATTENDANCE_DATA = generateAttendance();

const WEEKLY_TREND = [
  { day: "Mon", present: 82, absent: 18 },
  { day: "Tue", present: 75, absent: 25 },
  { day: "Wed", present: 88, absent: 12 },
  { day: "Thu", present: 70, absent: 30 },
  { day: "Fri", present: 65, absent: 35 },
];

const MONTHLY_TREND = [
  { month: "Jan", rate: 84 }, { month: "Feb", rate: 79 }, { month: "Mar", rate: 81 },
  { month: "Apr", rate: 76 }, { month: "May", rate: 83 }, { month: "Jun", rate: 88 },
];

// ---- STYLES ----
const injectStyles = () => {
  const style = document.createElement("style");
  style.textContent = `
    @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600;700&family=Inter:wght@400;500;600&display=swap');
    
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body { background: #0A0F1E; color: #F9FAFB; font-family: 'Inter', sans-serif; }
    
    ::-webkit-scrollbar { width: 6px; }
    ::-webkit-scrollbar-track { background: #0A0F1E; }
    ::-webkit-scrollbar-thumb { background: #1F2937; border-radius: 3px; }

    @keyframes pulse-ring {
      0% { transform: scale(0.8); opacity: 1; }
      100% { transform: scale(2.2); opacity: 0; }
    }
    @keyframes pulse-dot {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.15); }
    }
    @keyframes slide-in {
      from { opacity: 0; transform: translateY(16px); }
      to { opacity: 1; transform: translateY(0); }
    }
    @keyframes shimmer {
      0% { background-position: -200% 0; }
      100% { background-position: 200% 0; }
    }
    @keyframes glow {
      0%, 100% { box-shadow: 0 0 20px rgba(79,70,229,0.3); }
      50% { box-shadow: 0 0 40px rgba(79,70,229,0.6); }
    }
    @keyframes fadeIn {
      from { opacity: 0; } to { opacity: 1; }
    }
    @keyframes spin {
      from { transform: rotate(0deg); } to { transform: rotate(360deg); }
    }
    
    .slide-in { animation: slide-in 0.4s ease forwards; }
    .fade-in { animation: fadeIn 0.3s ease forwards; }

    .nav-item {
      display: flex; align-items: center; gap: 10px;
      padding: 10px 14px; border-radius: 10px; cursor: pointer;
      transition: all 0.2s; font-size: 14px; font-weight: 500;
      color: #6B7280; border: none; background: none; width: 100%;
    }
    .nav-item:hover { background: #1F2937; color: #F9FAFB; }
    .nav-item.active { background: #1E1B4B; color: #818CF8; }
    
    .stat-card {
      background: #111827; border: 1px solid #1F2937; border-radius: 16px;
      padding: 20px; transition: all 0.25s; cursor: default;
    }
    .stat-card:hover { border-color: #374151; transform: translateY(-2px); }
    
    .glass {
      background: rgba(17,24,39,0.8);
      backdrop-filter: blur(12px);
      border: 1px solid #1F2937;
    }
    
    .badge {
      display: inline-flex; align-items: center; gap: 4px;
      padding: 3px 10px; border-radius: 20px; font-size: 11px; font-weight: 600;
    }
    .badge-green { background: rgba(16,185,129,0.15); color: #10B981; }
    .badge-amber { background: rgba(245,158,11,0.15); color: #F59E0B; }
    .badge-red { background: rgba(239,68,68,0.15); color: #EF4444; }
    .badge-indigo { background: rgba(79,70,229,0.15); color: #818CF8; }

    .btn-primary {
      background: linear-gradient(135deg, #4F46E5, #06B6D4);
      color: white; border: none; padding: 10px 20px; border-radius: 10px;
      font-size: 13px; font-weight: 600; cursor: pointer; transition: all 0.2s;
      font-family: 'Inter', sans-serif;
    }
    .btn-primary:hover { opacity: 0.9; transform: translateY(-1px); }
    .btn-secondary {
      background: #1F2937; color: #9CA3AF; border: 1px solid #374151;
      padding: 9px 18px; border-radius: 10px; font-size: 13px;
      font-weight: 500; cursor: pointer; transition: all 0.2s;
      font-family: 'Inter', sans-serif;
    }
    .btn-secondary:hover { background: #374151; color: #F9FAFB; }

    .progress-bar {
      height: 6px; border-radius: 3px; background: #1F2937; overflow: hidden;
    }
    .progress-fill {
      height: 100%; border-radius: 3px;
      transition: width 1s ease;
    }

    .ai-chat-msg {
      max-width: 85%; padding: 10px 14px; border-radius: 14px;
      font-size: 13px; line-height: 1.6; animation: slide-in 0.3s ease;
    }
    .ai-msg { background: #1E1B4B; color: #C7D2FE; border-bottom-left-radius: 4px; }
    .user-msg { background: #1F2937; color: #F9FAFB; border-bottom-right-radius: 4px; margin-left: auto; }

    .table-row {
      display: grid; padding: 14px 16px; border-bottom: 1px solid #1F2937;
      align-items: center; transition: background 0.15s;
    }
    .table-row:hover { background: #111827; }

    input, select, textarea {
      background: #1F2937; border: 1px solid #374151; color: #F9FAFB;
      border-radius: 10px; padding: 10px 14px; font-size: 13px;
      font-family: 'Inter', sans-serif; outline: none; transition: border 0.2s;
    }
    input:focus, select:focus, textarea:focus { border-color: #4F46E5; }

    .tooltip {
      position: relative;
    }
    .tooltip:hover::after {
      content: attr(data-tip);
      position: absolute; bottom: 110%; left: 50%; transform: translateX(-50%);
      background: #1F2937; color: #F9FAFB; padding: 5px 10px; border-radius: 6px;
      font-size: 11px; white-space: nowrap; z-index: 100; border: 1px solid #374151;
    }

    .mini-chart-bar {
      display: flex; align-items: flex-end; gap: 3px; height: 40px;
    }
    .mini-bar {
      flex: 1; border-radius: 3px 3px 0 0; min-width: 8px;
      transition: height 0.5s ease;
    }
  `;
  document.head.appendChild(style);
};

// ---- ICONS (inline SVG) ----
const Icon = ({ name, size = 16, color = "currentColor" }) => {
  const icons = {
    dashboard: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><rect x="3" y="3" width="7" height="7" rx="1"/><rect x="14" y="3" width="7" height="7" rx="1"/><rect x="3" y="14" width="7" height="7" rx="1"/><rect x="14" y="14" width="7" height="7" rx="1"/></svg>,
    students: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>,
    attendance: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M9 11l3 3L22 4"/><path d="M21 12v7a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/></svg>,
    analytics: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>,
    ai: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><circle cx="12" cy="12" r="3"/><path d="M12 1v4M12 19v4M4.22 4.22l2.83 2.83M16.95 16.95l2.83 2.83M1 12h4M19 12h4M4.22 19.78l2.83-2.83M16.95 7.05l2.83-2.83"/></svg>,
    alert: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>,
    send: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><line x1="22" y1="2" x2="11" y2="13"/><polygon points="22 2 15 22 11 13 2 9 22 2"/></svg>,
    trend: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/></svg>,
    camera: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/><circle cx="12" cy="13" r="4"/></svg>,
    notification: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>,
    download: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>,
    qr: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/><rect x="14" y="14" width="2" height="2"/><rect x="19" y="14" width="2" height="2"/><rect x="14" y="19" width="2" height="2"/><rect x="19" y="19" width="2" height="2"/></svg>,
    star: <svg width={size} height={size} viewBox="0 0 24 24" fill={color} stroke={color} strokeWidth="1"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>,
    close: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>,
    refresh: <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth="2"><polyline points="23 4 23 10 17 10"/><polyline points="1 20 1 14 7 14"/><path d="M3.51 9a9 9 0 0 1 14.85-3.36L23 10M1 14l4.64 4.36A9 9 0 0 0 20.49 15"/></svg>,
  };
  return icons[name] || null;
};

// ---- LIVE PULSE DOT ----
const PulseDot = ({ color = "#10B981", size = 10 }) => (
  <div style={{ position: "relative", width: size, height: size }}>
    <div style={{
      position: "absolute", inset: 0, borderRadius: "50%", background: color,
      animation: "pulse-dot 2s ease-in-out infinite"
    }} />
    <div style={{
      position: "absolute", inset: 0, borderRadius: "50%", background: color,
      animation: "pulse-ring 2s ease-out infinite"
    }} />
  </div>
);

// ---- AVATAR ----
const Avatar = ({ initials, risk, size = 36 }) => {
  const colors = { low: ["#064E3B", "#10B981"], medium: ["#78350F", "#F59E0B"], high: ["#7F1D1D", "#EF4444"] };
  const [bg, fg] = colors[risk] || colors.low;
  return (
    <div style={{
      width: size, height: size, borderRadius: "50%", background: bg,
      color: fg, display: "flex", alignItems: "center", justifyContent: "center",
      fontSize: size * 0.33, fontWeight: 700, flexShrink: 0,
      fontFamily: "'Space Grotesk', sans-serif"
    }}>
      {initials}
    </div>
  );
};

// ---- MINI BAR CHART ----
const MiniBarChart = ({ data, color }) => (
  <div className="mini-chart-bar">
    {data.map((v, i) => (
      <div key={i} className="mini-bar" style={{
        height: `${v}%`, background: color, opacity: 0.7 + (i / data.length) * 0.3
      }} />
    ))}
  </div>
);

// ---- SPARKLINE ----
const Sparkline = ({ data, color = "#4F46E5", width = 80, height = 32 }) => {
  const max = Math.max(...data), min = Math.min(...data);
  const pts = data.map((v, i) => {
    const x = (i / (data.length - 1)) * width;
    const y = height - ((v - min) / (max - min || 1)) * height;
    return `${x},${y}`;
  }).join(" ");
  return (
    <svg width={width} height={height} style={{ overflow: "visible" }}>
      <polyline points={pts} fill="none" stroke={color} strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" />
      <circle cx={pts.split(" ").pop().split(",")[0]} cy={pts.split(" ").pop().split(",")[1]} r="3" fill={color} />
    </svg>
  );
};

// ---- DONUT CHART ----
const DonutChart = ({ percent, color, size = 70 }) => {
  const r = 28, cx = 35, cy = 35;
  const circ = 2 * Math.PI * r;
  const dash = (percent / 100) * circ;
  return (
    <svg width={size} height={size} viewBox="0 0 70 70">
      <circle cx={cx} cy={cy} r={r} fill="none" stroke="#1F2937" strokeWidth="6" />
      <circle cx={cx} cy={cy} r={r} fill="none" stroke={color} strokeWidth="6"
        strokeDasharray={`${dash} ${circ}`} strokeLinecap="round"
        transform="rotate(-90 35 35)" style={{ transition: "stroke-dasharray 1s ease" }} />
      <text x="35" y="39" textAnchor="middle" fontSize="13" fontWeight="700"
        fill="#F9FAFB" fontFamily="'Space Grotesk',sans-serif">{percent}%</text>
    </svg>
  );
};

// ---- STAT CARD ----
const StatCard = ({ label, value, sub, icon, color, trend, sparkData }) => (
  <div className="stat-card slide-in">
    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 12 }}>
      <div style={{ padding: 8, borderRadius: 10, background: color + "22" }}>
        <Icon name={icon} size={18} color={color} />
      </div>
      {sparkData && <Sparkline data={sparkData} color={color} />}
    </div>
    <div style={{ fontSize: 28, fontWeight: 700, fontFamily: "'Space Grotesk',sans-serif", color: "#F9FAFB", lineHeight: 1 }}>{value}</div>
    <div style={{ fontSize: 12, color: "#6B7280", marginTop: 4 }}>{label}</div>
    {sub && (
      <div style={{ display: "flex", alignItems: "center", gap: 4, marginTop: 8, fontSize: 12 }}>
        <span style={{ color: trend >= 0 ? "#10B981" : "#EF4444" }}>
          {trend >= 0 ? "↑" : "↓"} {Math.abs(trend)}%
        </span>
        <span style={{ color: "#6B7280" }}>{sub}</span>
      </div>
    )}
  </div>
);

// ---- AI CHAT PANEL ----
const AIAssistant = ({ onClose }) => {
  const [messages, setMessages] = useState([
    { role: "ai", text: "👋 Hi! I'm AttendAI, your intelligent attendance assistant. Ask me anything — dropout risks, attendance patterns, or generate reports!" }
  ]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const bottomRef = useRef(null);

  useEffect(() => { bottomRef.current?.scrollIntoView({ behavior: "smooth" }); }, [messages]);

  const QUICK = ["Who are at-risk students?", "Summarize today's attendance", "Which subject has lowest attendance?", "Generate weekly report"];

  const sendMessage = async (text) => {
    const q = text || input.trim();
    if (!q) return;
    setInput("");
    setMessages(m => [...m, { role: "user", text: q }]);
    setLoading(true);

    const context = `You are AttendAI, an expert student attendance analytics assistant embedded in a smart campus management system.
You have access to the following attendance data:
- 8 students in CSE Year 3
- 5 subjects: Data Structures, Machine Learning, DBMS, Networks, OS
- At-risk students (below 75%): Rohan Mehta (avg 58%), Karan Patel (avg 61%)
- Medium risk: Priya Nair (avg 72%), Meera Pillai (avg 71%), Ananya Roy (avg 70%)
- Good standing: Arjun Sharma (avg 88%), Sneha Iyer (avg 87%), Dev Anand (avg 89%)
- OS has the lowest average attendance at 63%
- Today's overall attendance: 78%
- Weekly trend: Monday 82%, Tuesday 75%, Wednesday 88%, Thursday 70%, Friday 65%
- AI predicts Rohan Mehta and Karan Patel may not meet 75% threshold this semester

Be concise, insightful, and actionable. Use emojis sparingly. Format key data clearly. Max 3-4 sentences unless a report is requested.`;

    try {
      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-6",
          max_tokens: 1000,
          system: context,
          messages: [
            ...messages.filter(m => m.role !== "ai" || messages.indexOf(m) > 0).map(m => ({
              role: m.role === "ai" ? "assistant" : "user",
              content: m.text
            })),
            { role: "user", content: q }
          ]
        })
      });
      const data = await res.json();
      const reply = data.content?.[0]?.text || "I couldn't process that. Please try again.";
      setMessages(m => [...m, { role: "ai", text: reply }]);
    } catch {
      setMessages(m => [...m, { role: "ai", text: "⚠️ Connection error. Please check your API key and try again." }]);
    }
    setLoading(false);
  };

  return (
    <div style={{
      position: "fixed", right: 20, bottom: 20, width: 380, height: 560,
      background: "#0D1117", border: "1px solid #1F2937", borderRadius: 20,
      display: "flex", flexDirection: "column", zIndex: 1000,
      boxShadow: "0 25px 60px rgba(0,0,0,0.6), 0 0 40px rgba(79,70,229,0.15)",
      animation: "slide-in 0.35s ease"
    }}>
      {/* Header */}
      <div style={{
        padding: "16px 18px", borderBottom: "1px solid #1F2937",
        background: "linear-gradient(135deg, #1E1B4B, #0A0F1E)",
        borderRadius: "20px 20px 0 0", display: "flex", alignItems: "center", justifyContent: "space-between"
      }}>
        <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
          <div style={{ padding: 8, background: "rgba(79,70,229,0.3)", borderRadius: 10 }}>
            <Icon name="ai" size={16} color="#818CF8" />
          </div>
          <div>
            <div style={{ fontSize: 14, fontWeight: 700, fontFamily: "'Space Grotesk',sans-serif", color: "#F9FAFB" }}>AttendAI Assistant</div>
            <div style={{ fontSize: 11, color: "#6B7280", display: "flex", alignItems: "center", gap: 4 }}>
              <PulseDot color="#10B981" size={6} /> Live · Powered by Claude
            </div>
          </div>
        </div>
        <button onClick={onClose} style={{ background: "none", border: "none", cursor: "pointer", color: "#6B7280" }}>
          <Icon name="close" size={16} />
        </button>
      </div>

      {/* Messages */}
      <div style={{ flex: 1, overflowY: "auto", padding: 16, display: "flex", flexDirection: "column", gap: 10 }}>
        {messages.map((m, i) => (
          <div key={i} style={{ display: "flex", justifyContent: m.role === "user" ? "flex-end" : "flex-start" }}>
            <div className={`ai-chat-msg ${m.role === "ai" ? "ai-msg" : "user-msg"}`}
              style={{ whiteSpace: "pre-wra

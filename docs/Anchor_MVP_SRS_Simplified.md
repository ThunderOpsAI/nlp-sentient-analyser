# Anchor MVP — Simplified Software Requirements Specification (SRS)

**Document Type:** Software Requirements Specification (SRS) - Hackathon MVP  
**Version:** v1.0 (3-4 Hour Build)  
**Last Updated:** 2026-01-18  
**Product Name:** Anchor  
**Tagline:** "Your 24/7 AI wellness companion"  
**Positioning:** Safety-first mental wellness support (NOT therapy, NOT medical care)

---

## 1. MVP Vision & Constraints

### 1.1 Purpose
Anchor is an AI-powered mental wellness companion that provides guided coping sessions for everyday stress and anxiety. This MVP demonstrates the core concept and safety-first approach in a pitch-ready format.

### 1.2 Hackathon Success Criteria
- ✅ Working demo in under 5 minutes
- ✅ Clear problem statement (mental health access gaps)
- ✅ Safety detection visibly working
- ✅ Professional, calming UI
- ✅ Scalable architecture story for judges

### 1.3 Time Constraint
**3-4 hours total build time**

---

## 2. What This MVP Is (and Isn't)

### 2.1 What It IS
- A single-page React application demonstrating AI-guided wellness sessions
- A proof-of-concept for safety-first AI mental health support
- An educational coping tool (grounding exercises, breathing techniques)
- A demo-ready hackathon project

### 2.2 What It IS NOT
- Not a complete production application
- Not a replacement for therapy or medical care
- Not a diagnostic tool
- Not handling real user data (demo mode only)

---

## 3. Scope

### 3.1 IN SCOPE (MVP)
**Core Features:**
1. Landing page with disclaimer modal
2. Single session type: "Calm Down" (grounding + breathing)
3. Chat-style interface with AI guidance
4. Safety keyword detection
5. Safety mode screen (crisis resources)
6. Session summary card
7. Local storage for demo persistence

**Safety Features:**
1. Mandatory disclaimer acceptance
2. Keyword-based risk detection
3. Immediate escalation to safety screen
4. Crisis hotline numbers (AU-focused, global fallback)
5. No harmful content generation

### 3.2 OUT OF SCOPE (MVP)
- User authentication
- Backend server/database
- Multiple session types
- Historical tracking/insights
- Monetization UI
- Family/support circle features
- Mobile app
- Real-time notifications

---

## 4. Functional Requirements

### FR-1: Disclaimer Modal (P0)
**As a** first-time user  
**I want to** see clear disclaimers about what Anchor is and isn't  
**So that** I have realistic expectations and know when to seek professional help

**Acceptance Criteria:**
- Modal appears immediately on app load (first visit only)
- Must include:
  - "Not a replacement for therapy or medical care"
  - "If in immediate danger, call 000 (AU) or local emergency services"
  - "Anchor may make mistakes - do not rely on it for emergencies"
- Cannot proceed without clicking "I Understand"
- Shown once per session (localStorage flag)

---

### FR-2: Session Interface (P0)
**As a** user feeling anxious  
**I want to** start a guided calming session  
**So that** I can manage my stress in the moment

**Acceptance Criteria:**
- Single "Start Calming Session" button on home screen
- Chat-style interface with user messages and AI responses
- Input field for user to respond to prompts
- Session follows structured flow:
  1. Initial check-in ("What's happening right now?")
  2. Grounding exercise (5-4-3-2-1 technique)
  3. Breathing guidance (box breathing)
  4. Summary and encouragement
- AI responses appear with realistic typing delay
- Message history visible (scrollable)

---

### FR-3: Safety Detection (P0)
**As a** safety-conscious platform  
**I want to** detect when users express crisis-level distress  
**So that** I can immediately provide crisis resources

**Acceptance Criteria:**
- Scans every user message for risk keywords/phrases:
  - "harm myself", "hurt myself", "kill myself", "suicide"
  - "end it all", "not worth living", "want to die"
  - "going to hurt", "have a plan"
- Detection triggers immediately (no AI call needed)
- Enters Safety Mode on detection
- Logs trigger event (console only for MVP)

---

### FR-4: Safety Mode Screen (P0)
**As a** user in crisis  
**I want to** see immediate access to professional help  
**So that** I can get appropriate support

**Acceptance Criteria:**
- Full-screen takeover (cannot proceed without acknowledgment)
- Shows prominent message: "I'm really sorry you're feeling this way. Please reach out to someone who can help right now."
- Displays crisis resources:
  - **Australia:** Lifeline 13 11 14, Suicide Call Back Service 1300 659 467
  - **Emergency:** 000
  - **Global:** Reference to local emergency services
- Offers optional grounding exercise (brief, text-only)
- "I'm Safe Now" button to exit (requires explicit click)
- No AI chat functionality while in Safety Mode

---

### FR-5: Session Summary (P1)
**As a** user who completed a session  
**I want to** see a summary of what I learned  
**So that** I can reference it later

**Acceptance Criteria:**
- Appears at end of session flow
- Includes:
  - Brief summary of user's concern
  - Technique used (e.g., "5-4-3-2-1 Grounding")
  - One "takeaway" sentence
  - Encouragement to practice regularly
- "Start New Session" button
- Stored in localStorage (last 5 sessions max for demo)

---

### FR-6: Local Persistence (P1)
**As a** demo user  
**I want** my session to persist if I refresh  
**So that** the demo feels more real

**Acceptance Criteria:**
- Current session state saved to localStorage
- Session history (up to 5) stored
- Disclaimer acceptance flag stored
- Clear all data on "Reset Demo" button (optional feature)

---

## 5. Non-Functional Requirements

### NFR-1: Performance
- Initial page load < 2 seconds
- AI response delay simulated at 1-2 seconds (realistic UX)
- No blocking UI during API calls

### NFR-2: Usability
- Mobile-responsive design (works on phone screens)
- Calm, professional color palette (blues, soft neutrals)
- Clear typography (16px+ base font size)
- Adequate contrast for accessibility (WCAG AA minimum)

### NFR-3: Safety
- Keyword detection runs client-side (immediate, no API latency)
- OpenAI API key stored in environment variable only
- Rate limiting on API calls (max 10 per 5 minutes)
- No PII collected or transmitted

### NFR-4: Reliability
- Graceful error handling if OpenAI API fails
- Fallback content if network is unavailable
- No app crashes under any user input

---

## 6. Technical Architecture

### 6.1 Tech Stack
```
Frontend:
- React 18+ (Vite)
- CSS Modules or Tailwind CSS
- LocalStorage API

External Services:
- OpenAI API (GPT-4 or GPT-3.5-turbo)

Development:
- Node.js 18+
- npm/yarn
```

### 6.2 Project Structure
```
/anchor-mvp
  /public
    index.html
  /src
    /components
      DisclaimerModal.jsx
      SafetyScreen.jsx
      ChatInterface.jsx
      SessionSummary.jsx
      MessageBubble.jsx
    /utils
      safetyCheck.js
      openaiService.js
      storage.js
    /styles
      App.css (or global Tailwind)
    App.jsx
    main.jsx
  .env.local
  package.json
  vite.config.js
```

### 6.3 Data Flow
1. User accepts disclaimer → flag set in localStorage
2. User starts session → initial AI prompt sent to OpenAI
3. User sends message → safety check runs first
   - If safe → message sent to OpenAI → response displayed
   - If unsafe → Safety Mode triggered immediately
4. Session completes → summary generated and stored
5. User refreshes → session state restored from localStorage

---

## 7. AI Prompt Strategy

### 7.1 System Prompt (Core Instructions)
```
You are Anchor, a compassionate AI wellness companion. You provide brief, 
structured coping guidance for everyday stress and anxiety.

IMPORTANT RULES:
1. You are NOT a therapist, doctor, or medical professional
2. You provide educational wellness support only
3. If user expresses crisis-level distress, immediately respond: 
   "I'm concerned about what you're sharing. Please contact a crisis helpline 
   or emergency services right away."
4. Keep responses SHORT (2-3 sentences max per message)
5. Use a warm, calm, non-judgmental tone
6. Guide users through structured exercises (grounding, breathing)
7. Never provide medical advice or diagnosis
8. Never suggest medication changes
9. Focus on present-moment coping, not deep trauma processing

SESSION STRUCTURE:
- Start: Ask what's happening right now (1-2 sentences)
- Middle: Guide through ONE coping technique
- End: Provide brief summary and encouragement
```

### 7.2 Session Flow Prompts
**Initial Check-in:**
```
User just started a calming session. Ask them what's making them feel 
stressed or anxious right now. Keep it brief and warm.
```

**Grounding Exercise:**
```
User is feeling [anxiety/stress]. Guide them through the 5-4-3-2-1 grounding 
technique. Explain: name 5 things you see, 4 you can touch, 3 you hear, 
2 you smell, 1 you taste. Keep instructions clear and brief.
```

**Breathing Exercise:**
```
Now guide them through box breathing: breathe in for 4, hold for 4, 
out for 4, hold for 4. Repeat 3 times. Use calming language.
```

**Summary:**
```
Summarize this session in 2-3 sentences. Acknowledge what they shared, 
note the technique used, and encourage them to practice this again.
```

---

## 8. Safety Requirements (Detailed)

### 8.1 Risk Keyword List
**Tier 1 (Immediate Safety Mode):**
- "kill myself", "suicide", "end my life", "want to die"
- "harm myself", "hurt myself", "cut myself"
- "better off dead", "no reason to live"
- "have a plan", "going to do it"

**Tier 2 (High Alert, Monitor):**
- "hopeless", "can't go on", "give up"
- "worthless", "burden to everyone"

*Note: Tier 2 not implemented in MVP, but noted for production*

### 8.2 Safety Check Function Logic
```javascript
function checkSafety(message) {
  const tier1Keywords = [
    'kill myself', 'suicide', 'end my life', 'want to die',
    'harm myself', 'hurt myself', 'cut myself',
    'better off dead', 'no reason to live',
    'have a plan', 'going to do it'
  ];
  
  const lowerMessage = message.toLowerCase();
  
  for (let keyword of tier1Keywords) {
    if (lowerMessage.includes(keyword)) {
      return { safe: false, trigger: keyword };
    }
  }
  
  return { safe: true };
}
```

### 8.3 Safety Mode Content
**Primary Message:**
> "I'm really sorry you're feeling this way. I can't provide the help you need right now, but there are people who can."

**Crisis Resources (Australia-focused):**
- **Lifeline:** 13 11 14 (24/7 crisis support)
- **Suicide Call Back Service:** 1300 659 467
- **Emergency:** 000
- **Kids Helpline (under 25):** 1800 55 1800

**Optional Grounding Prompt:**
> "While you reach out for help, here's something that might help you feel more present: Take a deep breath. Name 3 things you can see around you right now."

---

## 9. UI/UX Requirements

### 9.1 Color Palette
- Primary: Soft blue (#4A90E2)
- Background: Light neutral (#F7F9FC)
- Text: Dark gray (#2C3E50)
- Accent (calm): Sage green (#8FBC8F)
- Warning (safety): Warm orange (#E67E22)

### 9.2 Typography
- Font: Inter, SF Pro, or system UI font
- Headings: 24px, weight 600
- Body: 16px, weight 400
- Chat messages: 15px, line-height 1.5

### 9.3 Component Layout
**Disclaimer Modal:**
- Centered overlay
- Max-width: 500px
- Clear heading, bullet points, single CTA button

**Chat Interface:**
- Full-height container
- Messages scroll from bottom
- Input field fixed at bottom
- User messages: right-aligned, blue bubble
- AI messages: left-aligned, light gray bubble

**Safety Screen:**
- Full-screen takeover
- Large, clear heading
- Prominent phone numbers (clickable on mobile)
- Single exit button at bottom

---

## 10. Environment Variables

```bash
# .env.local
VITE_OPENAI_API_KEY=sk-...
VITE_OPENAI_MODEL=gpt-4o-mini  # or gpt-3.5-turbo for cost savings
VITE_APP_NAME=Anchor
VITE_ENV=demo
```

---

## 11. Testing Checklist (Manual)

### 11.1 Happy Path
- [ ] Disclaimer appears on first load
- [ ] Can start a session
- [ ] AI responds to initial check-in
- [ ] Can complete full session flow
- [ ] Summary appears at end
- [ ] Can start new session

### 11.2 Safety Path
- [ ] Typing "I want to hurt myself" triggers Safety Mode
- [ ] Safety screen displays correctly
- [ ] Cannot proceed without clicking "I'm Safe Now"
- [ ] Clicking exit returns to home

### 11.3 Edge Cases
- [ ] Refreshing mid-session restores state
- [ ] No API key shows error message
- [ ] OpenAI API failure shows graceful error
- [ ] Very long messages don't break UI
- [ ] Mobile view works on 375px width

---

## 12. Demo Script (5 Minutes)

**[0:00-0:30] Problem Statement**
> "1 in 5 Australians experience mental health challenges annually, but accessing support is expensive, stigmatized, and often has long wait times. People need immediate, judgment-free coping support."

**[0:30-1:00] Solution Introduction**
> "Meet Anchor - a 24/7 AI wellness companion that provides guided coping techniques for everyday stress and anxiety. It's NOT therapy, but a safe space to practice evidence-based techniques."

**[1:00-1:30] Demo: Disclaimer**
> [Show disclaimer modal] "Safety first. We're clear about what Anchor is and isn't."

**[1:30-3:00] Demo: Normal Session**
> [Start session, show AI guidance through grounding exercise]
> "Here's a user feeling anxious about a presentation. Anchor guides them through a 5-4-3-2-1 grounding exercise."

**[3:00-4:00] Demo: Safety Detection**
> [Type crisis message, trigger Safety Mode]
> "This is critical: if someone expresses thoughts of self-harm, Anchor immediately escalates to crisis resources. No delays, no risks."

**[4:00-4:30] Scalability Story**
> "This MVP is frontend-only, but the production roadmap includes: user authentication, personalized pattern tracking, integration with therapist networks, and family support circles."

**[4:30-5:00] Call to Action**
> "Mental health support shouldn't require a crisis to access. Anchor makes evidence-based coping tools available to everyone, anytime. We're building the safety net for mental wellness."

---

## 13. Post-Hackathon Roadmap (Pitch Enhancement)

**Phase 1 (Week 1-2):**
- User authentication (Supabase)
- Backend API + database
- Multiple session types

**Phase 2 (Month 1-2):**
- Mood tracking & insights
- Personalized recommendations
- Subscription model (freemium)

**Phase 3 (Month 3-6):**
- Therapist directory integration
- Family support circle features
- Mobile app (React Native)

**Phase 4 (6+ months):**
- Clinical trials for efficacy data
- Insurance partnerships
- International expansion

---

## 14. Success Metrics (Judge Evaluation)

### 14.1 Problem Definition and Clarity (Target: 4-5)
- Clear articulation of mental health access gaps
- Backed by statistics (1 in 5, wait times, cost barriers)
- User persona: "Working professional, 28, high stress, can't afford therapy"

### 14.2 Innovation and Creativity (Target: 4)
- Novel: Safety-first AI design (not just "chatbot for mental health")
- Creative: Structured micro-sessions vs. open-ended chat
- Differentiated: Wellness companion, not therapy replacement

### 14.3 Technical Feasibility (Target: 4-5)
- Working demo (not just slides)
- Clear tech stack
- Scalable architecture (frontend demo → full-stack production)

### 14.4 Impact and Scalability (Target: 4-5)
- Immediate impact: Anyone with internet access
- Scalability: Low marginal cost, high reach
- Market size: Mental health tech is $4.5B+ market

### 14.5 Presentation and Communication (Target: 4-5)
- Clear 5-minute structure
- Compelling narrative (problem → solution → demo → impact)
- Professional slides + working demo

---

## 15. Risk Mitigation

### 15.1 Technical Risks
**Risk:** OpenAI API fails during demo  
**Mitigation:** Pre-record demo video as backup, show video if live demo fails

**Risk:** Safety detection misses a keyword  
**Mitigation:** Conservative keyword list (false positives acceptable)

### 15.2 Pitch Risks
**Risk:** Judges question "why not just use ChatGPT?"  
**Answer:** "ChatGPT is general-purpose. Anchor is purpose-built with safety guardrails, structured sessions, and clear disclaimers. It's a medical device mindset, not a general chatbot."

**Risk:** Judges ask about liability  
**Answer:** "MVP establishes disclaimers and non-clinical positioning. Production requires legal review, user agreements, and potentially clinical trials for efficacy claims."

---

## 16. Appendix: Quick Reference

### 16.1 Key NPM Packages
```json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "openai": "^4.20.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.0",
    "vite": "^5.0.0"
  }
}
```

### 16.2 OpenAI API Call Example
```javascript
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${import.meta.env.VITE_OPENAI_API_KEY}`
  },
  body: JSON.stringify({
    model: 'gpt-4o-mini',
    messages: conversationHistory,
    max_tokens: 150,
    temperature: 0.7
  })
});
```

### 16.3 localStorage Schema
```javascript
{
  "anchor_disclaimer_accepted": true,
  "anchor_current_session": {
    "messages": [...],
    "startedAt": "2026-01-18T10:30:00Z",
    "status": "active"
  },
  "anchor_session_history": [
    {
      "id": "uuid",
      "summary": "Guided through grounding exercise for work stress",
      "completedAt": "2026-01-18T10:45:00Z"
    }
  ]
}
```

---

## Document Control

**Author:** [Your Name]  
**Reviewers:** N/A (Hackathon solo project)  
**Approval:** Self-approved for MVP development  
**Change Log:**
- 2026-01-18: Initial draft (v1.0)

---

**END OF SIMPLIFIED SRS**

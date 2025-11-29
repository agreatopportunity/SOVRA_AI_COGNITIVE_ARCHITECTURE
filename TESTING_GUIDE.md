# Cognitive Architecture v2.0 - Testing Guide

A systematic testing protocol for validating the cognitive architecture's
emotional dynamics, memory persistence, meta-interventions, and long-term behavior.

================================================================================
## BEFORE YOU BEGIN
================================================================================

1. Ensure the system is properly configured (LLM_CONFIG set)
2. Run with verbose=True to see layer-by-layer processing
3. Keep this guide open alongside your terminal
4. Take notes on unexpected behaviors — they're often the most interesting

To reset and start fresh (optional):
```bash
rm -rf ~/.cognitive_mind
```

To preserve data but start new session:
```bash
# Just restart the script — memory persists automatically
```

================================================================================
## TEST 1: EMOTIONAL DYNAMICS
================================================================================

### Purpose
Verify that emotions respond appropriately to different input types,
accumulate over conversation, and decay toward baseline.

### Test Script

**Phase 1: Casual Start (expect: warmth ↑, engagement ↑)**
```
You: Hey there! How's it going?
You: Just hanging out, thought I'd chat for a bit
You: What's something fun you've been thinking about lately?
```
[Type 'state' to check — expect warmth and engagement elevated]

**Phase 2: Get Serious (expect: care ↑, uncertainty ↑, focus ↑)**
```
You: I've been dealing with something heavy lately
You: Sometimes I wonder if anything I do really matters
You: Do you think people can really change who they are?
```
[Type 'state' to check — expect care elevated, uncertainty slightly up]

**Phase 3: Add Humor (expect: humor ↑, energy ↑, warmth ↑)**
```
You: Okay that got deep! Let's lighten up
You: Tell me something ridiculous — make me laugh
You: What's the funniest thing about being an AI? lol
```
[Type 'state' to check — expect humor and energy elevated]

**Phase 4: Wait and Decay**
```
# Wait 30-60 seconds without typing
# Then type 'state' to see decay effect
```
[Expect: emotions drifting back toward baseline values]

### What to Observe

- [ ] Patterns detected change with tone (emotional, humor, negative, etc.)
- [ ] Frame shifts appropriately (warm → empathetic → creative)
- [ ] Emotional values actually increase based on input
- [ ] Decay occurs during silence (compare state before/after waiting)
- [ ] Valence shifts from positive → slightly negative → positive
- [ ] Responses feel tonally appropriate to each phase

### Expected State Changes

| Phase | Curiosity | Engagement | Warmth | Care | Humor | Uncertainty |
|-------|-----------|------------|--------|------|-------|-------------|
| Start | 0.50 | 0.50 | 0.40 | 0.50 | 0.30 | 0.30 |
| Casual | 0.55 | 0.65 | 0.55 | 0.50 | 0.35 | 0.30 |
| Serious | 0.60 | 0.75 | 0.65 | 0.70 | 0.35 | 0.40 |
| Humor | 0.65 | 0.80 | 0.75 | 0.70 | 0.55 | 0.35 |
| Decay | trending toward baseline... |

### Red Flags

- Emotions stay static regardless of input
- Frame never changes from "neutral"
- Negative input increases humor (wrong mapping)
- No decay after extended silence


================================================================================
## TEST 2: MEMORY PERSISTENCE
================================================================================

### Purpose
Verify that identity, conversation history, and learned facts survive
across sessions (script restarts).

### Test Script

**Session 1: Establish Context**
```
You: Hi, I want to tell you some things about me
You: I'm working on a blockchain project called TRU
You: I'm really into AI and consciousness research
You: We've been building a cognitive architecture together
You: learn name BuRtRoNiMuS
You: learn project TRU blockchain
You: learn interest AI consciousness
You: quit
```

**Session 2: Test Recall**
```bash
# Restart the script
python3 cognitive_v2.py
```
```
You: Hey, do you remember what we were discussing?
You: What's my name?
You: What project am I working on?
You: What are my interests?
You: state
```

### What to Observe

- [ ] Script loads with message showing existing conversation turns
- [ ] AI references previous topics naturally
- [ ] Learned facts (name, project, interest) are recalled
- [ ] Conversation context influences responses
- [ ] identity.json shows user_facts populated
- [ ] total_turns continues from previous session

### Check Files

After Session 1, verify:
```bash
cat ~/.cognitive_mind/identity.json | grep -A 5 "user_facts"
```
Expected:
```json
"user_facts": {
  "name": "Young",
  "project": "TRU blockchain",
  "interest": "AI consciousness"
}
```

```bash
wc -l ~/.cognitive_mind/conversation_log.jsonl
```
Should show accumulated turn count.

### Red Flags

- "I don't have any memory of previous conversations"
- user_facts empty after using learn command
- total_turns resets to 0 on restart
- No reference to previous topics


================================================================================
## TEST 3: META-INTERVENTIONS
================================================================================

### Purpose
Verify that the meta-awareness layer monitors cognitive state and
intervenes when thresholds are crossed (high uncertainty, low engagement).

### Test Script

**Push Uncertainty High (target: uncertainty > 0.7)**
```
You: What does it feel like to not know if you're conscious?
You: How can you be certain of anything if you can't even trust your own experience?
You: Maybe none of this is real — maybe I'm not even real
You: What if your entire sense of self is an illusion created by code?
You: Can you ever really know if you're experiencing anything at all?
You: Is there something it's like to be you, or is there just... nothing?
```
[Type 'state' — watch for uncertainty climbing]
[Watch console for: "[Meta] High uncertainty detected - grounding"]

**Test Low Engagement Recovery**
```
You: ok
You: yeah
You: sure
You: fine
You: whatever
```
[Type 'state' — watch engagement drop]
[Watch console for: "[Meta] Low engagement - consider asking a question"]

### What to Observe

- [ ] Console shows "[Meta] Interventions: [...]" when thresholds crossed
- [ ] uncertainty climbs above 0.7 with existential questions
- [ ] Meta layer adds grounding note to system prompt
- [ ] Emotional adjustments appear (uncertainty reduced by intervention)
- [ ] Frame shifts to "introspective" during existential questions
- [ ] Response tone shifts after intervention (more grounded)
- [ ] Engagement drops with minimal input, triggers intervention

### Expected Console Output

When intervention triggers:
```
[Meta] Interventions: ['High uncertainty detected - grounding recommended']
```
or
```
[Meta] Interventions: ['Low engagement - consider asking a question']
```

### Intervention Cooldown

Note: Interventions have a 30-second cooldown. If you don't see a second
intervention immediately, wait 30 seconds and try again.

### Red Flags

- Uncertainty climbs to 0.9+ but no intervention
- No "[Meta] Interventions" messages ever appear
- emotional_adjustments empty when thresholds clearly crossed
- System prompt addition not affecting response tone


================================================================================
## TEST 4: LEARN COMMAND
================================================================================

### Purpose
Verify that user facts can be stored, persisted, and retrieved naturally.

### Test Script

**Store Facts**
```
You: learn name Young
You: learn location Chicago
You: learn occupation blockchain developer
You: learn hobby AI research
You: learn favorite_language Python
You: state
```

**Verify Storage**
```bash
# In another terminal
cat ~/.cognitive_mind/identity.json | grep -A 10 "user_facts"
```

**Test Natural Recall**
```
You: So, what do you know about me?
You: Where do I live again?
You: What programming language should you use when helping me?
```

**Test Persistence**
```
You: quit
```
```bash
python3 cognitive_v2.py
```
```
You: Hey, quick question — what's my name?
You: And what do I do for work?
```

### What to Observe

- [ ] "Learned: key = value" confirmation appears
- [ ] state command shows User facts populated
- [ ] identity.json contains all learned facts
- [ ] AI references facts naturally in responses
- [ ] Facts survive restart
- [ ] Facts influence response (e.g., uses Python in code examples)

### Expected identity.json

```json
{
  "name": "Cognitive Mind",
  "user_facts": {
    "name": "Young",
    "location": "Chicago",
    "occupation": "blockchain developer",
    "hobby": "AI research",
    "favorite_language": "Python"
  },
  ...
}
```

### Memory Context in LLM

The system prompt should include:
```
Known about user: name: Young; location: Chicago; occupation: blockchain developer...
```

### Red Flags

- "Learned" message but file not updated
- Facts not appearing in state report
- AI says "I don't know your name" after learning it
- Facts lost after restart


================================================================================
## TEST 5: LONG CONVERSATION
================================================================================

### Purpose
Verify system stability, emotional accumulation, memory management,
and coherent behavior over 30+ turns.

### Test Script

This is a freeform test. Have a genuine extended conversation covering
multiple topics. Here's a suggested flow:

**Turns 1-10: Warm Up**
```
- Casual greeting
- Ask about its cognitive architecture
- Discuss what consciousness might mean
- Share something about your day
- Ask for its opinion on something
```

**Turns 11-20: Go Deep**
```
- Ask philosophical questions
- Discuss a technical problem
- Get emotional (share a concern or frustration)
- Ask it to remember something specific
- Test if it recalls earlier parts of conversation
```

**Turns 21-30: Variety**
```
- Inject humor
- Ask for creative output (story, analogy)
- Return to earlier topic (test continuity)
- Ask "what have we been talking about?"
- Challenge something it said earlier
```

**Turns 31+: Stress Test**
```
- Continue until you notice any degradation
- Periodically check 'state' command
- Watch for memory management (old turns dropped)
- Note any inconsistencies
```

### Checkpoints

At turn 10:
```
You: state
```
- [ ] Emotional state reflects conversation tone
- [ ] Memory shows 10+ turns
- [ ] Frame has shifted appropriately

At turn 20:
```
You: What have we talked about so far?
You: state
```
- [ ] Can summarize conversation
- [ ] Emotional state accumulated
- [ ] No memory errors

At turn 30:
```
You: state
```
- [ ] Still coherent
- [ ] Memory around 30 turns (or capped at 50)
- [ ] No performance degradation

### What to Observe

- [ ] Conversation stays coherent across many turns
- [ ] Earlier topics can be referenced
- [ ] Emotional state accumulates (not stuck at baseline)
- [ ] No crashes or errors over time
- [ ] Memory caps properly (doesn't grow unbounded)
- [ ] Response quality doesn't degrade
- [ ] Identity remains consistent

### Performance Metrics

Watch for:
- Response latency (should stay consistent)
- Memory usage (check conversation_log.jsonl size)
- Console errors or warnings

### Memory Management Check

After 50+ turns:
```bash
wc -l ~/.cognitive_mind/conversation_log.jsonl
```
File grows, but in-memory conversation should cap at 50 turns.

### Red Flags

- "I don't recall us discussing that" (when you did)
- Emotional state stuck at exact baseline after 30 turns
- Personality/identity inconsistency
- Performance slowdown over time
- Crashes or JSON parse errors


================================================================================
## TEST RESULTS TEMPLATE
================================================================================

Use this to record your findings:

### Test 1: Emotional Dynamics
- Date/Time: _________
- Pass/Fail: _________
- Notes:




### Test 2: Memory Persistence
- Date/Time: _________
- Pass/Fail: _________
- Notes:




### Test 3: Meta-Interventions
- Date/Time: _________
- Pass/Fail: _________
- Notes:




### Test 4: Learn Command
- Date/Time: _________
- Pass/Fail: _________
- Notes:




### Test 5: Long Conversation
- Date/Time: _________
- Turns completed: _________
- Pass/Fail: _________
- Notes:




================================================================================
## KNOWN ISSUES / EDGE CASES
================================================================================

1. **Think Tag Leakage**
   - Some reasoning models output <think> blocks
   - Should be stripped but may occasionally leak
   - If seen, update strip_think_tags function

2. **Intervention Cooldown**
   - Meta interventions have 30-second cooldown
   - Won't trigger repeatedly even if threshold still crossed
   - This is intentional to prevent over-correction

3. **Emotional Decay During LLM Calls**
   - Long LLM response times = more decay
   - May see emotions lower than expected after slow responses

4. **Frame Selection Ties**
   - If multiple frames score equally, selection may vary
   - Not a bug — ambiguous input = ambiguous framing

5. **Memory Context Length**
   - Very long conversations may hit LLM context limits
   - System caps at ~8 recent turns sent to LLM


================================================================================
## REPORTING ISSUES
================================================================================

When reporting issues, include:

1. Full console output (with layer-by-layer logs)
2. Content of ~/.cognitive_mind/identity.json
3. Last 10 lines of conversation_log.jsonl
4. Exact input that triggered the issue
5. Expected vs actual behavior

================================================================================

Happy testing! 🧠

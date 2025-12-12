# Mixo Ads - Backend Engineering Assignment

## 🎯 Overview

You've inherited a backend service that syncs campaign performance data from an Ad Platform API to our database. The previous developer got it "working" for a single test account but left before handling production scale.

**The problem:** The code works... sometimes. It breaks frequently, has no error handling, and won't scale past 5 clients.

**Your job:** Fix it, restructure it, and document it properly.

**Time estimate:** 4-5 hours  
**Deadline:** 72 hours from receiving this assignment

---

## �️ Tools & Resources

**You're encouraged to use ANY tools you'd normally use at work:**
- ✅ LLMs (ChatGPT, Claude, GitHub Copilot, etc.)
- ✅ Stack Overflow, documentation, tutorials
- ✅ Any libraries or frameworks you prefer

We're testing your problem-solving and understanding, not memorization.

**Important:** After submission, selected candidates will have a **technical review call** (30-45 minutes) where you'll walk through your solution and discuss your design decisions.

---

## �🚀 Setup Instructions

### Prerequisites
- Node.js 18+ and npm
- Git

### Step 1: Clone this repository

```bash
git clone <repository-url>
cd mixo-backend-task
```

### Step 2: Install dependencies

```bash
# Install main app dependencies
npm install

# Install mock API dependencies
cd mock-api
npm install
cd ..
```

### Step 3: Set up environment variables

```bash
cp .env.example .env
```

The `.env` file contains the credentials needed for the mock API.

### Step 4: Start the Mock API

In one terminal window:

```bash
cd mock-api
npm start
```

The mock API will run on http://localhost:3001

You should see:
```
🚀 Mock Ad Platform API Server
📍 Server running on: http://localhost:3001
🔑 Valid credentials: admin@mixoads.com / SuperSecret123!
```

### Step 5: Run the broken sync script

In another terminal window:

```bash
npm start
```

**Watch it fail.** 🔥

You'll see various issues:
- Only 10 campaigns synced (should be 100)
- Credentials logged in plain text
- Random failures
- Slow sequential processing
- And more...

---

## 🐛 What's Wrong?

The code has **multiple critical issues**. Some are obvious, some are subtle. You need to find and fix them.

**Categories of issues:**
- ❌ Authentication problems (hardcoded, logged, insecure)
- ❌ Rate limiting not handled (API has 10 req/min limit)
- ❌ No error handling (crashes on failures)
- ❌ No retry logic (transient failures kill the job)
- ❌ No timeout handling (hangs forever)
- ❌ Pagination broken (only gets first 10 of 100 campaigns)
- ❌ Performance problems (sequential, slow)
- ❌ Code structure disasters (god function, mixed concerns)
- ❌ Database issues (duplicates, SQL injection, connection leaks)
- ❌ Security problems (exposed credentials, stack traces)

---

## ✅ What You Need To Do

### 1. Fix the Critical Bugs (Must Do)

- **Authentication:** Move credentials to environment variables, stop logging them
- **Rate Limiting:** Handle 429 responses with proper backoff
- **Error Handling:** Add try-catch blocks, don't crash on failures
- **Retry Logic:** Implement exponential backoff for transient failures
- **Timeout Handling:** Add timeouts so requests don't hang forever
- **Pagination:** Fetch all 100 campaigns, not just the first 10
- **Database:** Fix duplicates, use parameterized queries, proper connection management

### 2. Improve Code Structure (Should Do)

- **Break apart the god function** - It's 100+ lines doing everything
- **Separate concerns:**
  - Auth management (separate module)
  - API client (separate module)
  - Database operations (separate module)
  - Business logic (sync service)
- **Remove copy-pasted code**
- **Make it testable** - separate logic from I/O
- **Clean up naming** - use descriptive variable/function names
- **Add proper logging** - structured logs without sensitive data

### 3. Document Everything (Must Do)

Complete the `SUBMISSION.md` file (template provided) with:

**Part A: What Was Broken**
- List 5-7 major issues you found
- Explain why each was a problem
- Be specific about the impact

**Part B: How You Fixed It**
- Explain your approach to each fix
- Why did you structure it this way?
- What trade-offs did you make?
- Link to specific code changes

**Part C: How to Run It**
- Clear setup instructions
- What should happen when it works?
- How to test it?

**Part D: Production Considerations**
- What would you add before deploying to production?
- What monitoring/alerting would you set up?
- What could still go wrong?
- How would you scale to 100+ clients?

---

## 🎮 The Mock API

The mock API simulates a real ad platform with realistic constraints:

**Authentication:**
- Use Basic auth (base64 email:password) to get Bearer token
- Tokens expire after 1 hour
- Credentials: `admin@mixoads.com` / `SuperSecret123!`

**Rate Limits:**
- 10 requests per minute
- Returns 429 with `retry-after` header if exceeded

**Reliability Issues:**
- Random 503 errors (20% of requests) - should retry
- Random timeouts (10% of requests) - should timeout
- Slow sync endpoint (2 seconds) - should handle

**Data:**
- 100 total campaigns
- Paginated: 10 campaigns per page
- Must fetch all 10 pages to get complete data

See `mock-api/README.md` for full API documentation.

---

## 📤 Submission Instructions

### When you're done:

1. **Fork this repository** to your GitHub account

2. **Clone your fork** and make your changes
   ```bash
   git clone https://github.com/YOUR_USERNAME/mixoads-backend-assignment.git
   cd mixoads-backend-assignment
   ```

3. **Commit your changes** with clear, logical commits
   ```bash
   git add .
   git commit -m "Fix authentication and rate limiting"
   git commit -m "Refactor into separate modules"
   git commit -m "Add comprehensive error handling"
   ```

4. **Complete SUBMISSION.md** with all required sections

5. **Push to your fork**
   ```bash
   git push origin main
   ```

6. **Open a Pull Request** to the original repository
   - Title: `[Your Name] - Backend Assignment Submission`
   - Description: Brief summary of your approach

### Your submission should include:

- **All code changes** with clear commit messages
- **SUBMISSION.md** fully completed
- **Working code** that can be tested
- **Pull Request** opened against the original repository

---

## 📊 What We're Evaluating

### Must Have (70% of score):
- ✅ Fixed critical operational bugs
- ✅ System works reliably (we'll test it)
- ✅ Clear, comprehensive documentation
- ✅ Code is readable and organized

### Nice to Have (30% of score):
- 🌟 Significantly improved code structure
- 🌟 Added thoughtful logging/monitoring
- 🌟 Considered edge cases we didn't mention
- 🌟 Documentation shows deep production thinking

### Red Flags (Auto-Reject):
- ❌ Only fixed syntax errors, missed systemic issues
- ❌ Fixed one thing but broke another
- ❌ Can't explain decisions in documentation
- ❌ Over-engineered (10 classes for simple sync)
- ❌ Didn't test - code doesn't actually work

---

## 💡 Tips

- **Test your fixes** - Run it multiple times, make sure it works
- **Document as you go** - Easier than writing everything at the end
- **Don't over-engineer** - Practical fixes > perfect architecture
- **Use any resources** - Google, Stack Overflow, LLMs, docs - whatever helps
- **Ask if stuck** - Email hari@mixoads.com, we'll respond quickly

**Remember:** We want to see how you think, not just that you can write code. Your documentation is as important as your code.

---

## ❓ Questions?

Email: hari@mixoads.com

---

**Good luck! We're excited to see your work.** 🚀

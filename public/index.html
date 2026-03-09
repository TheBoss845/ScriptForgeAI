require('dotenv').config();

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const Stripe = require('stripe');
const path = require('path');
const crypto = require('crypto');
const nodemailer = require('nodemailer');

const app = express();

// ─────────────────────────────────────────────
// Stripe webhook needs raw body BEFORE json middleware
// ─────────────────────────────────────────────
app.use('/api/payments/webhook', express.raw({ type: 'application/json' }));

app.use(cors());
app.use(express.json());
app.use(express.static(path.join(__dirname, '..', 'public')));

// ─────────────────────────────────────────────
// DATABASE CONNECTION
// ─────────────────────────────────────────────
mongoose
    .connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/scriptforge')
    .then(() => console.log('✓ MongoDB Connected'))
    .catch((err) => console.error('MongoDB Error:', err.message));

// ─────────────────────────────────────────────
// SCHEMAS
// ─────────────────────────────────────────────
const userSchema = new mongoose.Schema({
    firstName: String,
    lastName: String,
    email: { type: String, unique: true, required: true },
    password: { type: String, required: true },
    plan: { type: String, enum: ['starter', 'creator', 'studio'], default: 'starter' },
    scriptsUsed: { type: Number, default: 0 },
    scriptsLimit: { type: Number, default: 5 },
    scriptsResetDate: { type: Date, default: () => nextMonthDate() },
    stripeCustomerId: String,
    stripeSubscriptionId: String,
    // Password reset
    resetPasswordToken: String,
    resetPasswordExpires: Date,
    // Email verification
    emailVerified: { type: Boolean, default: false },
    emailVerifyToken: String,
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now },
});

const scriptSchema = new mongoose.Schema({
    userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    title: String,
    topic: String,
    hook: String,
    intro: String,
    sections: [{ heading: String, content: String }],
    cta: String,
    outro: String,
    hookOptions: [String],
    titles: [String],
    thumbnailIdea: String,
    format: String,
    audience: String,
    tone: Number,
    createdAt: { type: Date, default: Date.now },
});

const User = mongoose.model('User', userSchema);
const Script = mongoose.model('Script', scriptSchema);

// ─────────────────────────────────────────────
// HELPERS
// ─────────────────────────────────────────────
function nextMonthDate() {
    const d = new Date();
    d.setMonth(d.getMonth() + 1);
    d.setDate(1);
    d.setHours(0, 0, 0, 0);
    return d;
}

// Resets scriptsUsed to 0 if the reset date has passed
async function checkAndResetScripts(user) {
    if (new Date() >= new Date(user.scriptsResetDate)) {
        user.scriptsUsed = 0;
        user.scriptsResetDate = nextMonthDate();
        await user.save();
    }
}

// ─────────────────────────────────────────────
// EMAIL (Nodemailer via Gmail App Password or SMTP)
// ─────────────────────────────────────────────
function createTransporter() {
    // Works with Gmail: enable 2FA → generate App Password → put it in .env
    // Or use any SMTP service like Resend or Mailgun
    if (!process.env.EMAIL_USER || !process.env.EMAIL_PASS) {
        console.warn('⚠️  EMAIL_USER / EMAIL_PASS not set — emails will be skipped');
        return null;
    }
    return nodemailer.createTransport({
        service: process.env.EMAIL_SERVICE || 'gmail',
        auth: {
            user: process.env.EMAIL_USER,
            pass: process.env.EMAIL_PASS,
        },
    });
}

async function sendEmail(to, subject, htmlBody) {
    const transporter = createTransporter();
    if (!transporter) return; // silently skip if not configured
    try {
        await transporter.sendMail({
            from: `"ScriptForge" <${process.env.EMAIL_USER}>`,
            to,
            subject,
            html: htmlBody,
        });
        console.log(`✓ Email sent to ${to}`);
    } catch (err) {
        console.error('Email send error:', err.message);
    }
}

function welcomeEmailHtml(firstName, plan) {
    return `
    <div style="font-family:sans-serif;max-width:560px;margin:0 auto;background:#05080f;color:#eef0f5;padding:32px;border-radius:12px">
      <div style="font-size:1.6rem;font-weight:900;color:#c8ff00;letter-spacing:2px;margin-bottom:24px">SCRIPTFORGE</div>
      <h2 style="margin:0 0 12px">Welcome, ${firstName}! 🎬</h2>
      <p style="color:#8892a4;line-height:1.7">Your <strong style="color:#c8ff00">${plan.charAt(0).toUpperCase()+plan.slice(1)} plan</strong> is active and ready to go.</p>
      <p style="color:#8892a4;line-height:1.7">Head back to ScriptForge and start generating your first script. It takes about 8 minutes from idea to a complete, ready-to-film YouTube script.</p>
      <a href="${process.env.FRONTEND_URL || 'https://scriptforge.app'}" style="display:inline-block;background:#c8ff00;color:#05080f;font-weight:800;padding:12px 28px;border-radius:8px;text-decoration:none;margin-top:16px">Open ScriptForge →</a>
      <p style="color:#5a6478;font-size:.78rem;margin-top:32px">You're receiving this because you signed up at ScriptForge. Questions? Reply to this email.</p>
    </div>`;
}

function resetPasswordEmailHtml(firstName, resetUrl) {
    return `
    <div style="font-family:sans-serif;max-width:560px;margin:0 auto;background:#05080f;color:#eef0f5;padding:32px;border-radius:12px">
      <div style="font-size:1.6rem;font-weight:900;color:#c8ff00;letter-spacing:2px;margin-bottom:24px">SCRIPTFORGE</div>
      <h2 style="margin:0 0 12px">Reset your password</h2>
      <p style="color:#8892a4;line-height:1.7">Hi ${firstName}, we got a request to reset your ScriptForge password.</p>
      <p style="color:#8892a4;line-height:1.7">Click the button below. This link expires in <strong style="color:#eef0f5">1 hour</strong>.</p>
      <a href="${resetUrl}" style="display:inline-block;background:#c8ff00;color:#05080f;font-weight:800;padding:12px 28px;border-radius:8px;text-decoration:none;margin-top:16px">Reset Password →</a>
      <p style="color:#5a6478;font-size:.78rem;margin-top:32px">If you didn't request this, ignore this email — your password won't change.</p>
    </div>`;
}

function upgradeEmailHtml(firstName, plan) {
    const perks = {
        creator: '✦ Unlimited scripts · 3 hook options · YouTube Analyzer · Thumbnail ideas',
        studio:  '✦ Everything in Creator · Team seats · API access · Priority generation',
    };
    return `
    <div style="font-family:sans-serif;max-width:560px;margin:0 auto;background:#05080f;color:#eef0f5;padding:32px;border-radius:12px">
      <div style="font-size:1.6rem;font-weight:900;color:#c8ff00;letter-spacing:2px;margin-bottom:24px">SCRIPTFORGE</div>
      <h2 style="margin:0 0 12px">You're on ${plan.charAt(0).toUpperCase()+plan.slice(1)} now 🚀</h2>
      <p style="color:#8892a4;line-height:1.7">Hi ${firstName}, your upgrade is live. Here's what you unlocked:</p>
      <p style="color:#c8ff00;line-height:1.9;font-size:.9rem">${perks[plan] || ''}</p>
      <a href="${process.env.FRONTEND_URL || 'https://scriptforge.app'}" style="display:inline-block;background:#c8ff00;color:#05080f;font-weight:800;padding:12px 28px;border-radius:8px;text-decoration:none;margin-top:16px">Start Creating →</a>
    </div>`;
}

// ─────────────────────────────────────────────
// STRIPE + ANTHROPIC INIT
// ─────────────────────────────────────────────
const stripe = process.env.STRIPE_SECRET_KEY ? new Stripe(process.env.STRIPE_SECRET_KEY) : null;
if (!stripe) console.warn('⚠️  STRIPE_SECRET_KEY not set — payments will be disabled');
if (!process.env.ANTHROPIC_API_KEY) console.warn('⚠️  ANTHROPIC_API_KEY not set — AI endpoints will fail');

// ─────────────────────────────────────────────
// MIDDLEWARE
// ─────────────────────────────────────────────
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No token provided' });
    jwt.verify(token, process.env.JWT_SECRET || 'your_secret_key', (err, user) => {
        if (err) return res.status(403).json({ error: 'Invalid token' });
        req.user = user;
        next();
    });
};

// ─────────────────────────────────────────────
// ANTHROPIC HELPER
// ─────────────────────────────────────────────
async function callAnthropic(messages, maxTokens = 2000, tools = null) {
    if (!process.env.ANTHROPIC_API_KEY) throw new Error('ANTHROPIC_API_KEY is not configured');

    const body = {
        model: process.env.ANTHROPIC_MODEL || 'claude-sonnet-4-5',
        max_tokens: maxTokens,
        messages,
    };
    if (tools) body.tools = tools;

    const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
            'content-type': 'application/json',
            'x-api-key': process.env.ANTHROPIC_API_KEY,
            'anthropic-version': '2023-06-01',
        },
        body: JSON.stringify(body),
    });

    if (!response.ok) {
        const text = await response.text();
        throw new Error(`Anthropic API error (${response.status}): ${text}`);
    }

    const data = await response.json();
    // Extract all text blocks (handles tool use responses too)
    return (data.content || [])
        .filter(b => b.type === 'text')
        .map(b => b.text)
        .join('');
}

// ─────────────────────────────────────────────
// ── HEALTH CHECK
// ─────────────────────────────────────────────
app.get('/api/health', (req, res) => res.json({ ok: true, time: new Date() }));

// ─────────────────────────────────────────────
// ── AUTH — SIGNUP
// ─────────────────────────────────────────────
app.post('/api/auth/signup', async (req, res) => {
    try {
        const { firstName, lastName, email, password, plan = 'starter' } = req.body;
        if (!email || !password) return res.status(400).json({ error: 'Email and password are required' });
        if (password.length < 8) return res.status(400).json({ error: 'Password must be at least 8 characters' });

        const existingUser = await User.findOne({ email });
        if (existingUser) return res.status(400).json({ error: 'An account with that email already exists' });

        const hashedPassword = await bcrypt.hash(password, 10);
        const emailVerifyToken = crypto.randomBytes(32).toString('hex');

        const newUser = new User({
            firstName,
            lastName,
            email,
            password: hashedPassword,
            plan,
            scriptsLimit: plan === 'starter' ? 5 : 999999,
            emailVerifyToken,
        });
        await newUser.save();

        // Send welcome + verify email
        const verifyUrl = `${process.env.FRONTEND_URL || ''}/api/auth/verify-email?token=${emailVerifyToken}&email=${encodeURIComponent(email)}`;
        await sendEmail(email, 'Welcome to ScriptForge — verify your email', welcomeEmailHtml(firstName || 'Creator', plan) + `
          <p style="margin-top:16px;color:#8892a4;font-size:.85rem">Also, please <a href="${verifyUrl}" style="color:#c8ff00">verify your email address</a> to unlock all features.</p>`);

        const token = jwt.sign(
            { id: newUser._id, email: newUser.email, plan: newUser.plan },
            process.env.JWT_SECRET || 'your_secret_key',
            { expiresIn: '30d' }
        );

        res.status(201).json({
            token,
            user: { id: newUser._id, firstName, email, plan: newUser.plan, scriptsUsed: 0, scriptsLimit: newUser.scriptsLimit },
        });
    } catch (error) {
        console.error('Signup error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── AUTH — EMAIL VERIFICATION
// ─────────────────────────────────────────────
app.get('/api/auth/verify-email', async (req, res) => {
    try {
        const { token, email } = req.query;
        const user = await User.findOne({ email, emailVerifyToken: token });
        if (!user) return res.status(400).send('<h2>Invalid or expired verification link.</h2>');
        user.emailVerified = true;
        user.emailVerifyToken = undefined;
        await user.save();
        res.redirect((process.env.FRONTEND_URL || '/') + '?verified=true');
    } catch (error) {
        res.status(500).send('<h2>Verification failed. Please try again.</h2>');
    }
});

// ─────────────────────────────────────────────
// ── AUTH — LOGIN
// ─────────────────────────────────────────────
app.post('/api/auth/login', async (req, res) => {
    try {
        const { email, password } = req.body;
        if (!email || !password) return res.status(400).json({ error: 'Email and password are required' });

        const user = await User.findOne({ email });
        if (!user) return res.status(404).json({ error: 'No account found with that email' });

        const isMatch = await bcrypt.compare(password, user.password);
        if (!isMatch) return res.status(401).json({ error: 'Incorrect password' });

        // Auto-reset scripts if the month has rolled over
        await checkAndResetScripts(user);

        const token = jwt.sign(
            { id: user._id, email: user.email, plan: user.plan },
            process.env.JWT_SECRET || 'your_secret_key',
            { expiresIn: '30d' }
        );

        res.json({
            token,
            user: {
                id: user._id,
                firstName: user.firstName,
                email: user.email,
                plan: user.plan,
                scriptsUsed: user.scriptsUsed,
                scriptsLimit: user.scriptsLimit,
                scriptsResetDate: user.scriptsResetDate,
            },
        });
    } catch (error) {
        console.error('Login error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── AUTH — FORGOT PASSWORD (sends reset email)
// ─────────────────────────────────────────────
app.post('/api/auth/forgot-password', async (req, res) => {
    try {
        const { email } = req.body;
        if (!email) return res.status(400).json({ error: 'Email is required' });

        const user = await User.findOne({ email });
        // Always return success — don't reveal if email exists
        if (!user) return res.json({ message: 'If that email exists, a reset link has been sent.' });

        const token = crypto.randomBytes(32).toString('hex');
        user.resetPasswordToken = token;
        user.resetPasswordExpires = Date.now() + 60 * 60 * 1000; // 1 hour
        await user.save();

        const resetUrl = `${process.env.FRONTEND_URL || ''}?reset_token=${token}&email=${encodeURIComponent(email)}`;
        await sendEmail(email, 'Reset your ScriptForge password', resetPasswordEmailHtml(user.firstName || 'there', resetUrl));

        res.json({ message: 'If that email exists, a reset link has been sent.' });
    } catch (error) {
        console.error('Forgot password error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── AUTH — RESET PASSWORD (uses token from email)
// ─────────────────────────────────────────────
app.post('/api/auth/reset-password', async (req, res) => {
    try {
        const { email, token, newPassword } = req.body;
        if (!email || !token || !newPassword) return res.status(400).json({ error: 'All fields are required' });
        if (newPassword.length < 8) return res.status(400).json({ error: 'Password must be at least 8 characters' });

        const user = await User.findOne({
            email,
            resetPasswordToken: token,
            resetPasswordExpires: { $gt: Date.now() },
        });

        if (!user) return res.status(400).json({ error: 'Reset link is invalid or has expired' });

        user.password = await bcrypt.hash(newPassword, 10);
        user.resetPasswordToken = undefined;
        user.resetPasswordExpires = undefined;
        await user.save();

        res.json({ message: 'Password reset successfully. You can now log in.' });
    } catch (error) {
        console.error('Reset password error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── USER — GET PROFILE
// ─────────────────────────────────────────────
app.get('/api/user/profile', authenticateToken, async (req, res) => {
    try {
        const user = await User.findById(req.user.id).select('-password -resetPasswordToken -emailVerifyToken');
        if (!user) return res.status(404).json({ error: 'User not found' });
        await checkAndResetScripts(user);
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── USER — UPDATE PROFILE
// ─────────────────────────────────────────────
app.put('/api/user/profile', authenticateToken, async (req, res) => {
    try {
        const { firstName, lastName } = req.body;
        const user = await User.findByIdAndUpdate(
            req.user.id,
            { firstName, lastName, updatedAt: new Date() },
            { new: true }
        ).select('-password');
        res.json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── USER — CHANGE PASSWORD (while logged in)
// ─────────────────────────────────────────────
app.post('/api/user/change-password', authenticateToken, async (req, res) => {
    try {
        const { currentPassword, newPassword } = req.body;
        if (!currentPassword || !newPassword) return res.status(400).json({ error: 'Both fields are required' });
        if (newPassword.length < 8) return res.status(400).json({ error: 'New password must be at least 8 characters' });

        const user = await User.findById(req.user.id);
        const isMatch = await bcrypt.compare(currentPassword, user.password);
        if (!isMatch) return res.status(401).json({ error: 'Current password is incorrect' });

        user.password = await bcrypt.hash(newPassword, 10);
        await user.save();
        res.json({ message: 'Password changed successfully' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── SCRIPTS — GENERATE
// ─────────────────────────────────────────────
app.post('/api/scripts/generate', authenticateToken, async (req, res) => {
    try {
        const { topic, format = 'listicle', audience = 'general', length = '8–12 min', tone = 3, context = '' } = req.body;
        if (!topic) return res.status(400).json({ error: 'Topic is required' });

        const user = await User.findById(req.user.id);
        if (!user) return res.status(404).json({ error: 'User not found' });

        // Check and auto-reset monthly counter
        await checkAndResetScripts(user);

        // Enforce plan limits
        if (user.plan === 'starter' && user.scriptsUsed >= user.scriptsLimit) {
            return res.status(403).json({ error: 'Monthly script limit reached. Upgrade to continue.' });
        }

        const toneDescriptions = ['calm & informative', 'conversational', 'balanced', 'energetic', 'high-octane viral'];
        const toneDesc = toneDescriptions[(tone - 1)] || 'balanced';
        const hookCount = user.plan === 'starter' ? 1 : 3;
        const titleCount = user.plan === 'starter' ? 3 : 5;
        const includeThumbnail = user.plan !== 'starter';

        const prompt = `You are a world-class YouTube scriptwriter. Return ONLY valid JSON, no markdown fences, no explanation.

{
  "hook": "2-3 sentence opening that stops scrollers cold",
  "hook_options": [${Array.from({length: hookCount}, (_,i) => `"hook option ${i+1}"`).join(',')}],
  "intro": "2-3 sentences after hook setting up the promise",
  "sections": [
    {"heading": "section title", "content": "4-6 sentences of vivid script content"}
  ],
  "cta": "2 compelling sentences for subscribe/comment",
  "outro": "warm closing line",
  "titles": [${Array.from({length: titleCount}, (_,i) => `"title ${i+1}"`).join(',')}]${includeThumbnail ? ',\n  "thumbnail_idea": "one concrete thumbnail sentence"' : ''}
}

Topic: ${topic}
Format: ${format}
Audience: ${audience}
Length: ${length}
Tone: ${toneDesc}
${context ? 'Context: ' + context : ''}

Write 5-7 sections minimum. Make hooks genuinely arresting. Titles must use proven YouTube patterns.`;

        const responseText = await callAnthropic([{ role: 'user', content: prompt }], 4000);
        const jsonMatch = responseText.match(/\{[\s\S]*\}/);
        if (!jsonMatch) throw new Error('Could not parse AI response. Please try again.');

        const scriptData = JSON.parse(jsonMatch[0]);

        // Save to database
        const script = new Script({
            userId: req.user.id,
            title: scriptData.titles?.[0] || topic,
            topic, format, audience, tone,
            hook: scriptData.hook,
            intro: scriptData.intro,
            sections: scriptData.sections || [],
            cta: scriptData.cta,
            outro: scriptData.outro,
            hookOptions: scriptData.hook_options || [scriptData.hook],
            titles: scriptData.titles || [],
            thumbnailIdea: scriptData.thumbnail_idea || '',
        });
        await script.save();

        // Increment usage counter
        user.scriptsUsed += 1;
        await user.save();

        res.json({
            script: { id: script._id, ...scriptData },
            scriptsUsed: user.scriptsUsed,
            scriptsLimit: user.scriptsLimit,
            scriptsRemaining: user.plan === 'starter' ? user.scriptsLimit - user.scriptsUsed : null,
        });
    } catch (error) {
        console.error('Script generation error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── SCRIPTS — GET ALL FOR USER
// ─────────────────────────────────────────────
app.get('/api/scripts', authenticateToken, async (req, res) => {
    try {
        const scripts = await Script.find({ userId: req.user.id })
            .sort({ createdAt: -1 })
            .select('title topic format createdAt'); // summary only
        res.json(scripts);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── SCRIPTS — GET ONE
// ─────────────────────────────────────────────
app.get('/api/scripts/:id', authenticateToken, async (req, res) => {
    try {
        const script = await Script.findById(req.params.id);
        if (!script) return res.status(404).json({ error: 'Script not found' });
        if (script.userId.toString() !== req.user.id) return res.status(403).json({ error: 'Unauthorized' });
        res.json(script);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── SCRIPTS — DELETE ONE
// ─────────────────────────────────────────────
app.delete('/api/scripts/:id', authenticateToken, async (req, res) => {
    try {
        const script = await Script.findById(req.params.id);
        if (!script) return res.status(404).json({ error: 'Script not found' });
        if (script.userId.toString() !== req.user.id) return res.status(403).json({ error: 'Unauthorized' });
        await script.deleteOne();
        res.json({ message: 'Script deleted' });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── TITLES — GENERATE
// ─────────────────────────────────────────────
app.post('/api/titles/generate', authenticateToken, async (req, res) => {
    try {
        const { topic, niche = 'general', styles = ['curiosity'] } = req.body;
        if (!topic) return res.status(400).json({ error: 'Topic is required' });

        const prompt = `Generate exactly 10 YouTube titles for: "${topic}"
Niche: ${niche}
Styles to use: ${styles.join(', ')}
Return ONLY a JSON array of 10 strings. No markdown, no explanation.
["title1","title2","title3","title4","title5","title6","title7","title8","title9","title10"]`;

        const responseText = await callAnthropic([{ role: 'user', content: prompt }], 800);
        const jsonMatch = responseText.match(/\[[\s\S]*\]/);
        if (!jsonMatch) throw new Error('Could not parse titles response');

        const titles = JSON.parse(jsonMatch[0]);
        res.json({ titles });
    } catch (error) {
        console.error('Title generation error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── YOUTUBE ANALYZER
// ─────────────────────────────────────────────
app.post('/api/analyzer/analyze', authenticateToken, async (req, res) => {
    try {
        const { url, aspects = ['hook', 'structure', 'title', 'audience'] } = req.body;
        if (!url) return res.status(400).json({ error: 'YouTube URL is required' });

        // Check plan — analyzer is Creator/Studio only
        const user = await User.findById(req.user.id);
        if (user.plan === 'starter') return res.status(403).json({ error: 'YouTube Analyzer requires Creator or Studio plan' });

        const prompt = `Search the web for this YouTube video: ${url}

Analyze the following aspects: ${aspects.join(', ')}

Return ONLY valid JSON:
{
  "video_title": "actual title of the video",
  "channel": "channel name",
  "topic": "core topic for making your own version",
  "format": "listicle|documentary|tutorial|review|story|educational",
  "audience": "description of target audience",
  "key_angles": ["angle1", "angle2", "angle3"],
  "hook_insight": "why the hook works",
  "improvement": "how to differentiate your version",
  "suggested_title": "improved title for your version",
  "full_analysis": "3-4 paragraph plain text deep dive on why this video works"
}`;

        const responseText = await callAnthropic(
            [{ role: 'user', content: prompt }],
            2000,
            [{ type: 'web_search_20250305', name: 'web_search' }]
        );

        const jsonMatch = responseText.match(/\{[\s\S]*\}/);
        if (!jsonMatch) throw new Error('Could not parse analyzer response');

        const analysis = JSON.parse(jsonMatch[0]);
        res.json({ analysis });
    } catch (error) {
        console.error('Analyzer error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── PAYMENTS — CREATE CHECKOUT SESSION
// ─────────────────────────────────────────────
app.post('/api/payments/checkout', authenticateToken, async (req, res) => {
    if (!stripe) return res.status(500).json({ error: 'Stripe is not configured' });

    try {
        const { plan } = req.body;
        const user = await User.findById(req.user.id);

        const plans = {
            starter: { priceId: process.env.STRIPE_PRICE_STARTER, name: 'Starter' },
            creator: { priceId: process.env.STRIPE_PRICE_CREATOR, name: 'Creator' },
            studio:  { priceId: process.env.STRIPE_PRICE_STUDIO,  name: 'Studio'  },
        };

        if (!plans[plan]) return res.status(400).json({ error: 'Invalid plan' });
        if (!plans[plan].priceId) return res.status(400).json({ error: `Stripe price ID for ${plan} is not configured` });

        const session = await stripe.checkout.sessions.create({
            customer_email: user.email,
            payment_method_types: ['card'],
            line_items: [{ price: plans[plan].priceId, quantity: 1 }],
            mode: 'subscription',
            success_url: `${process.env.FRONTEND_URL}/?session_id={CHECKOUT_SESSION_ID}&paid=${plan}`,
            cancel_url: `${process.env.FRONTEND_URL}/`,
            metadata: { userId: user._id.toString(), plan },
        });

        res.json({ sessionId: session.id, url: session.url });
    } catch (error) {
        console.error('Checkout error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── PAYMENTS — STRIPE WEBHOOK
// Stripe calls this automatically after successful payment
// to upgrade the user's plan in the database
// ─────────────────────────────────────────────
app.post('/api/payments/webhook', async (req, res) => {
    if (!stripe) return res.status(500).json({ error: 'Stripe not configured' });

    const sig = req.headers['stripe-signature'];
    let event;

    try {
        event = stripe.webhooks.constructEvent(
            req.body,
            sig,
            process.env.STRIPE_WEBHOOK_SECRET
        );
    } catch (err) {
        console.error('Webhook signature error:', err.message);
        return res.status(400).send(`Webhook Error: ${err.message}`);
    }

    try {
        if (event.type === 'checkout.session.completed') {
            const session = event.data.object;
            const userId = session.metadata?.userId;
            const plan = session.metadata?.plan;

            if (userId && plan) {
                const limits = { starter: 5, creator: 999999, studio: 999999 };
                const user = await User.findByIdAndUpdate(
                    userId,
                    {
                        plan,
                        scriptsLimit: limits[plan] ?? 5,
                        stripeCustomerId: session.customer,
                        stripeSubscriptionId: session.subscription,
                        updatedAt: new Date(),
                    },
                    { new: true }
                );

                if (user) {
                    // Send upgrade confirmation email
                    if (plan !== 'starter') {
                        await sendEmail(
                            user.email,
                            `You're now on ScriptForge ${plan.charAt(0).toUpperCase()+plan.slice(1)}!`,
                            upgradeEmailHtml(user.firstName || 'Creator', plan)
                        );
                    }
                    console.log(`✓ User ${userId} upgraded to ${plan}`);
                }
            }
        }

        // Handle subscription cancellations — downgrade to starter
        if (event.type === 'customer.subscription.deleted') {
            const subscription = event.data.object;
            const user = await User.findOne({ stripeSubscriptionId: subscription.id });
            if (user) {
                user.plan = 'starter';
                user.scriptsLimit = 5;
                user.scriptsUsed = 0;
                user.stripeSubscriptionId = undefined;
                await user.save();
                console.log(`✓ User ${user._id} downgraded to starter (subscription cancelled)`);
            }
        }

        res.json({ received: true });
    } catch (error) {
        console.error('Webhook handler error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// ── MONTHLY SCRIPT RESET
// This route is called by a cron job on the 1st of each month.
// On Railway: set up a cron job to hit POST /api/cron/reset-scripts
// with header Authorization: Bearer <CRON_SECRET>
// ─────────────────────────────────────────────
app.post('/api/cron/reset-scripts', async (req, res) => {
    const secret = req.headers['authorization']?.replace('Bearer ', '');
    if (secret !== process.env.CRON_SECRET) {
        return res.status(401).json({ error: 'Unauthorized' });
    }

    try {
        const now = new Date();
        // Reset all users whose reset date has passed
        const result = await User.updateMany(
            { scriptsResetDate: { $lte: now } },
            {
                $set: {
                    scriptsUsed: 0,
                    scriptsResetDate: nextMonthDate(),
                }
            }
        );
        console.log(`✓ Monthly reset: ${result.modifiedCount} users reset`);
        res.json({ reset: result.modifiedCount });
    } catch (error) {
        console.error('Cron reset error:', error);
        res.status(500).json({ error: error.message });
    }
});

// ─────────────────────────────────────────────
// FALLBACK — serve index.html for all non-API routes
// ─────────────────────────────────────────────
app.get('*', (req, res) => {
    if (!req.path.startsWith('/api')) {
        res.sendFile(path.join(__dirname, '..', 'public', 'index.html'));
    }
});

// ─────────────────────────────────────────────
// ERROR HANDLER
// ─────────────────────────────────────────────
app.use((err, req, res, next) => {
    console.error('Unhandled error:', err);
    res.status(500).json({ error: 'Internal server error' });
});

// ─────────────────────────────────────────────
// START SERVER
// ─────────────────────────────────────────────
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`✓ ScriptForge server running on port ${PORT}`);
    console.log(`  Environment: ${process.env.NODE_ENV || 'development'}`);
    console.log(`  MongoDB: ${process.env.MONGODB_URI ? 'configured' : 'using localhost'}`);
    console.log(`  Stripe: ${stripe ? 'configured' : 'NOT configured'}`);
    console.log(`  Anthropic: ${process.env.ANTHROPIC_API_KEY ? 'configured' : 'NOT configured'}`);
    console.log(`  Email: ${process.env.EMAIL_USER ? 'configured' : 'NOT configured'}`);
});

module.exports = app;

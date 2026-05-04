const nodemailer = require('nodemailer');
const { SocksProxyAgent } = require('socks-proxy-agent');

export default async function handler(req, res) {
  if (req.method !== 'POST') return res.status(405).json({ error: 'Method Not Allowed' });
  
  const { smtpConfig, mailData } = req.body;

  // Environment Variables for your AWS Rotating Proxy
  const mHost = process.env.MASTER_PROXY_HOST;
  const mPort = process.env.MASTER_PROXY_PORT;
  const mUser = process.env.MASTER_PROXY_USER;
  const mPass = process.env.MASTER_PROXY_PASS;

  // AGENT LOGIC: Added 'socks5h' for DNS masking (prevents Vercel IP leaks)
  let agent = null;
  if (mHost && mPort) {
    const proxyAuth = (mUser && mPass) ? `${mUser}:${mPass}@` : '';
    agent = new SocksProxyAgent(`socks5h://${proxyAuth}${mHost}:${mPort}`);
  }

  const transporter = nodemailer.createTransport({
    host: smtpConfig.host,
    port: parseInt(smtpConfig.port),
    secure: parseInt(smtpConfig.port) === 465, // True for 465, false for 587
    auth: { user: smtpConfig.user, pass: smtpConfig.pass },
    pool: true, // Re-use connections to avoid "Spammy" rapid re-connecting
    maxConnections: 3,
    maxMessages: 100,
    ...(agent && { agent }),
    tls: {
      rejectUnauthorized: false, // Ensures connection doesn't drop on cert issues
      minVersion: 'TLSv1.2'     // Required by Gmail/Yahoo in 2026
    }
  });

  try {
    const info = await transporter.sendMail({
      // CRITICAL: fromName must be clean, and sender address MUST match SMTP user
      from: `"${mailData.fromName}" <${smtpConfig.user}>`,
      to: mailData.to,
      subject: mailData.subject,
      html: mailData.html,
      list: {
        // One-Click Unsubscribe is REQUIRED for Inbox placement in 2026
        unsubscribe: {
          url: 'https://vpn-servic.site/unsubscribe',
          comment: 'Unsubscribe from this list'
        }
      },
      headers: {
        'X-Entity-Ref-ID': Math.random().toString(36).substring(7),
        'Precedence': 'bulk'
      }
    });

    return res.status(200).json({ success: true, messageId: info.messageId });
  } catch (error) {
    return res.status(500).json({ success: false, error: error.message });
  }
}

const { SocksProxyAgent } = require('socks-proxy-agent');
const axios = require('axios');

export default async function handler(req, res) {
    if (req.method !== 'POST') return res.status(405).json({ error: 'Method not allowed' });

    const host = process.env.MASTER_PROXY_HOST;
    const port = process.env.MASTER_PROXY_PORT;
    const user = process.env.MASTER_PROXY_USER;
    const pass = process.env.MASTER_PROXY_PASS;

    if (!host || !port) {
        return res.status(500).json({ 
            success: false, 
            error: "MISSING SERVER CONFIG: Add MASTER_PROXY_HOST and PORT to Vercel." 
        });
    }

    // Use socks5h to ensure DNS resolution happens on the AWS side
    const proxyUrl = `socks5h://${user}:${pass}@${host}:${port}`;
    
    try {
        const agent = new SocksProxyAgent(proxyUrl);
        const ipInfo = await axios.get('http://ip-api.com/json', { 
            httpAgent: agent, 
            httpsAgent: agent,
            timeout: 10000 
        });

        return res.status(200).json({
            success: true,
            ip: ipInfo.data.query,
            isp: ipInfo.data.isp,
            city: ipInfo.data.city,
            country: ipInfo.data.country
        });
    } catch (error) {
        return res.status(500).json({ 
            success: false, 
            error: "Proxy Failed: " + error.message 
        });
    }
}

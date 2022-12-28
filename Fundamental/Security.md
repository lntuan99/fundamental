1. Encryption
	- Size of cipher text:
		- Đối xứng: <= plain text
		- Bất đối xứng: >= plain text
	- Data size:
		- Đối xứng: transmit big data
		- Bất đối xứng: transmit small data
	- Key length:
		- Đối xứng: 128 hoặc 256 bit
		- Bất đối xứng: >= 2048 (RSA)
	- Number of keys:
		- Đối xứng: 1 key để khoá và giải
		- Bất đối xứng: 1 key để khoá và 1 key để giải
	- Bảo mật:
		- Bất đối xứng bảo mật tốt hơn
	- Tốc độ: bất đối xứng nhanh hơn
	- Thuật toán:
		- Đối xứng: AES-128-192-256, HMAC
		- Bất đối xứng: RSA, DSA, PKCS
2. Hack:
	- Injection: XSS, SQL injection
3. JWT:
	- gồm:
		- header
		- payload
		- signature (HMAC256(base64URLencode(header), base64URLencode(payload), secrect))
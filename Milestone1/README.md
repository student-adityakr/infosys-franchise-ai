# FranchiseOps AI

**Franchise Operations Intelligence Platform**  
Infosys Springboard Internship 7.0 · Batch 4 · Milestone 1

FranchiseOps AI is an educational prototype for a franchise operations intelligence platform. The Milestone 1 implementation focuses on secure user authentication. The franchise intelligence modules are planned for later milestones and are not represented as working features in this release.

## Milestone 1 features

- User sign-up with username, email, password confirmation, and a security question/answer.
- User login using username or email and password.
- Password recovery through either a security question or email OTP.
- Email OTP recovery with an expiry window and resend cooldown.
- Separate admin login.
- User dashboard that identifies the signed-in account and lists planned project modules.
- Password hashing, JWT-based session handling, and SQLite-backed account data.

> This is an educational internship prototype. Do not use real customer or employee records, and do not deploy it as a production authentication system without a security review.

## Screenshots

### User login
![Login page](screenshots/login.png)

### Account registration
![Sign-up page](screenshots/signup.png)

### User dashboard
![User dashboard](screenshots/user_dashboard.png)

### Admin login
![Admin login page](screenshots/admin_login.png)

### Password recovery — email OTP
![OTP recovery page](screenshots/password_recovery.png)

### Password recovery — security question
![Security question recovery](screenshots/security_recovery.png)

### Email OTP example
![Email OTP example with personal details obscured](screenshots/otp_email.png)

## Tech stack

- Python
- Streamlit
- SQLite
- PyJWT
- bcrypt
- pyngrok
- SMTP email delivery
- Google Colab for the provided notebook workflow

## Run the project in Google Colab

The provided notebook is designed to run in Google Colab and creates the application files in Google Drive.

1. Open `Agentic AI for Franchise Management System.ipynb` in Google Colab.
2. Run the notebook cells in order.
3. When prompted, mount Google Drive.
4. Add the required values to **Colab Secrets** before running the configuration cell.
5. Run the remaining cells to launch the Streamlit app and expose it through ngrok.
6. Open the generated public URL in your browser.

## Required configuration

Add these names to Colab Secrets. Keep the actual values private:

| Secret | Purpose |
|---|---|
| `JWT_SECRET` | Secret used to sign authentication tokens |
| `NGROK_AUTHTOKEN` | ngrok authentication token |
| `EMAIL_ADDRESS` | Sender email account for OTP delivery |
| `EMAIL_PASSWORD` | Email app password or SMTP credential |
| `ADMIN_USERNAME` | Admin account username |
| `ADMIN_PASSWORD` | Admin account password |

Use an email provider's app password where required. Never commit secrets, passwords, OTPs, database files, or real user records to GitHub.

## Project scope

The dashboard lists these planned modules:

- Outlet Performance
- Inventory Optimization
- Workforce Management
- Marketing Intelligence
- Compliance Audit (RAG)
- Customer Feedback
- Weekly Digest & Alerts

These are shown as **Planned** in the Milestone 1 dashboard; they are not implemented by this authentication milestone.

## Repository contents

- `Agentic AI for Franchise Management System.ipynb` — Google Colab notebook for setup and application generation.
- `screenshots/` — application screenshots used in this README.

## Security and demo notes

- Use only test accounts and non-sensitive data.
- Do not publish a live ngrok URL as a permanent production endpoint.
- Rotate any credential that has been accidentally exposed.
- The screenshots in this README are for demonstration; personal email/OTP details are obscured where visible.

## Author

Aditya Kumar  
B.Tech CSE (AI & ML) · UIET, Kurukshetra University

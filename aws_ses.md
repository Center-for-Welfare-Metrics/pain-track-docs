# SesPainTrack

This repository stores and publishes AWS SES email templates for the Pain Track application.

The application that sends the emails is the current `pain-app-server`.

- API URL: https://newpainappserver-840926881618.us-central1.run.app
- Website URL: https://www.pain-track.org/
- GitHub repository: https://github.com/Center-for-Welfare-Metrics/pain-app-server

## What This Project Does

Each template lives in its own folder under `src/` and has three parts:

- `templates/template.html`: the HTML email body
- `templates/text.txt`: the plain-text fallback body
- `src/update.ts`: a script that reads both template files and pushes them to AWS SES with `SES.updateTemplate`

The project does not send emails directly. It updates the template definitions that the current `pain-app-server` later uses when calling AWS SES templated email APIs.

## Integration with Pain Track

This repository is only responsible for managing SES templates.

The actual email sending happens in the current Pain Track backend:

- Application: `pain-app-server`
- API URL: https://newpainappserver-840926881618.us-central1.run.app
- Website URL: https://www.pain-track.org/
- GitHub repository: https://github.com/Center-for-Welfare-Metrics/pain-app-server

In practice, this means:

- This repository updates the templates stored in AWS SES
- The `pain-app-server` uses those templates when it needs to send emails to users

## How It Works

All four update scripts follow the same flow:

1. Load environment variables with `dotenv`.
2. Configure the AWS SDK for SES in region `us-east-1`.
3. Create an `AWS.SES` client using `AWS_ID` and `AWS_SECRET_KEY`.
4. Read the HTML and text template files from disk.
5. Call `SES.updateTemplate(...)` with a fixed SES template name and subject line.
6. Log either the AWS error or the AWS response.

Because the scripts call `updateTemplate`, the corresponding SES template must already exist in AWS. These scripts update existing templates; they do not create new ones.

## Project Layout

```text
src/
  contact/
    src/update.ts
    templates/template.html
    templates/text.txt
    templates/type.ts
  recovery-password/
    src/update.ts
    templates/template.html
    templates/text.txt
    templates/type.ts
  request-email-change-code/
    src/update.ts
    templates/template.html
    templates/text.txt
    templates/type.ts
  set-password/
    src/update.ts
    templates/template.html
    templates/text.txt
    templates/type.ts
```

## Templates

### `contact`

- SES template name: `CONTACT-FORM`
- Subject line: `Contact Form - Pain Track`
- Purpose: internal contact form message
- Placeholder data:

| Field     | Meaning                                  |
| --------- | ---------------------------------------- |
| `email`   | sender email address                     |
| `subject` | message subject                          |
| `message` | message body                             |
| `browser` | browser used to submit the form          |
| `os`      | operating system used to submit the form |
| `time`    | submission time                          |

### `recovery-password`

- SES template name: `RECOVERY-PASSWORD`
- Subject line: `Reset Password`
- Purpose: password reset email with action link
- Placeholder data:

| Field              | Meaning             |
| ------------------ | ------------------- |
| `name`             | recipient name      |
| `action_url`       | password reset URL  |
| `support_url`      | support/contact URL |
| `operating_system` | recipient device OS |
| `browser_name`     | recipient browser   |

### `request-email-change-code`

- SES template name: `REQUEST_EMAIL_CHANGE_CODE`
- Subject line: `Request Email Change - Pain Track`
- Purpose: verification code for changing email address
- Placeholder data:

| Field              | Meaning             |
| ------------------ | ------------------- |
| `name`             | recipient name      |
| `code`             | verification code   |
| `support_url`      | support/contact URL |
| `operating_system` | recipient device OS |
| `browser_name`     | recipient browser   |

### `set-password`

- SES template name: `SET_PASSWORD`
- Subject line: `Set Password - Pain Track`
- Purpose: verification code used during password setup flow
- Placeholder data:

| Field              | Meaning             |
| ------------------ | ------------------- |
| `name`             | recipient name      |
| `code`             | verification code   |
| `support_url`      | support/contact URL |
| `operating_system` | recipient device OS |
| `browser_name`     | recipient browser   |

## Environment Variables

The scripts rely on these environment variables:

| Variable         | Purpose               |
| ---------------- | --------------------- |
| `AWS_ID`         | AWS access key ID     |
| `AWS_SECRET_KEY` | AWS secret access key |

They are typed in `environment.d.ts` and loaded at runtime using `dotenv`.

## Available Scripts

Install dependencies first:

```bash
npm install
```

Build TypeScript:

```bash
npm run build
```

Push individual templates to AWS SES:

```bash
npm run send:template:contact
npm run send:template:recovery
npm run send:template:request-email-change
npm run send:template:set-password
```

These commands use `ts-node`, so they execute the TypeScript update scripts directly without a separate build step.

## Notes

- The TypeScript compiler is configured with `rootDir: src` and outputs compiled files to `build/`.
- The project uses the AWS SDK v2 package (`aws-sdk`).
- The HTML and text templates include Handlebars-style SES placeholders such as `{{name}}` and `{{code}}`.
- The `type.ts` files document the data shape expected when these SES templates are rendered by the application that sends the emails.

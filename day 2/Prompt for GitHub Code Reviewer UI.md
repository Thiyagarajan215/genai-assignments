# Prompt for Custom UI — AI GitHub Code Reviewer

## Phase 1 — Build the Complete UI

I am building an **AI-powered GitHub Code Reviewer** application.

Create a beautiful, professional, premium-looking user interface that feels like a real AI SaaS product.

### Requirements

1. Create a UI page where a user can submit a GitHub repository URL and see the review results on the same page.
2. Prepare the UI to connect to an n8n webhook, which will receive the repository URL and return the code-review response.
3. At this stage, create the HTML structure and CSS styling only. Do not add JavaScript functionality yet.

### Page content

- A polished header with an AI/code-review icon.
- Application title: **AI GitHub Code Reviewer**
- A short description explaining that the application reviews source code in a GitHub repository using Artificial Intelligence.
- A large input field where users can paste a GitHub repository URL, for example:
  - `https://github.com/owner/repository`
  - `https://github.com/owner/repository/tree/main/src`
- A modern **Review Repository** button.
- A **Review Results** section where the AI-generated code review will appear.
- Initial result-state text: **Ready to review your repository. Paste a GitHub URL to get started.**

### Design requirements

- Soft gradient background.
- Glassmorphism cards.
- Rounded corners and smooth shadows.
- Attractive, readable typography.
- Professional spacing and visual hierarchy.
- Polished hover effects and modern buttons.
- Responsive layout for desktop, tablet, and mobile.
- Clear visual distinction between the URL input, review action, loading state, errors, and results.
- Do not use Bootstrap or Tailwind CSS.
- Make the UI feel like a real AI developer tool, not a basic web page.

## Phase 2 — Connect the UI to n8n

The UI is already completed. Now connect it to my n8n backend while keeping the existing design unchanged.

When the user pastes a GitHub URL and clicks **Review Repository**:

1. Read the entered GitHub URL.
2. Validate that the input is not empty and looks like a GitHub repository or supported GitHub path URL.
3. Send the URL to the configured n8n webhook.
4. Wait for the webhook response.
5. Display a clear loading message while the repository review is running.
6. When the response arrives, display the AI review in the **Review Results** section on the same page.
7. Handle invalid URLs, network errors, timeouts, and unsuccessful webhook responses with friendly error messages.
8. If no URL is entered, ask the user to paste a GitHub URL.

### Integration and security requirements

- Use a clearly marked configuration setting or environment-backed server configuration for the n8n webhook URL; do not hard-code secrets into client-side code.
- Do not ask the user to paste a GitHub personal access token or API key into the UI.
- Do not expose credentials, webhook secrets, or other sensitive configuration in browser code or logs.
- If private repositories need to be supported, handle authentication securely on the backend. Do not imply that a repository can be reviewed without appropriate access.
- Send the repository URL using a documented request format agreed with the n8n workflow, and handle the response format explicitly.
- Do not invent the webhook URL or assume an undocumented payload/response schema. Use placeholders and clearly identify what must be configured if these details have not been provided.
- Keep the existing UI and layout unchanged; add only the functionality needed for the webhook integration.

## Phase 3 — Improve the User Experience

The application is working correctly. Improve the user experience without changing the backend integration or the core workflow.

Please add:

- An attractive loading animation while the review is in progress.
- Clear progress/status text, such as **Connecting to GitHub…**, **Reviewing repository…**, and **Preparing results…**, only if these states can be represented accurately by the backend workflow.
- Better button hover, pressed, disabled, and focus states.
- Smooth animations and transitions that do not distract from the review.
- A success message when the review completes.
- Friendly, actionable error messages if something goes wrong.
- A retry option after a failed review.
- Readable formatting for review findings, including headings, severity, file paths, line references, and suggested fixes when those details are returned by the backend.
- Accessible labels, keyboard navigation, and visible focus indicators.
- Preserve the current layout and do not change the backend integration.

## Phase 4 — Make It Feel Like a Premium AI SaaS Product

Enhance the design without changing the existing functionality, workflow, or backend integration.

Improve the application with:

- A beautiful gradient background.
- A subtle animated AI/code-review logo.
- A modern navigation bar.
- Premium cards and a polished dashboard layout.
- Consistent, attractive icons.
- Stylish primary and secondary buttons.
- Elegant typography and a professional color theme.
- Smooth, restrained animations.
- Responsive behavior for mobile and desktop.
- A clean results layout that makes findings easy to scan.
- Helpful empty, loading, success, and error states.
- A clear distinction between critical issues, warnings, suggestions, and positive observations when the review response provides that classification.

The final result should look and feel like a commercial AI developer tool.

Do not change the review workflow or backend integration. Focus on visual polish, accessibility, and user experience.

## Expected behavior and boundaries

- The user supplies a **GitHub URL**, not a Jira story key.
- The UI submits the URL and displays the response on the same page.
- The application must not claim to have reviewed code until a real backend response has been received.
- Do not fabricate findings, file paths, line numbers, or fixes.
- If the URL points to a private repository or a path the backend cannot access, explain that the repository may require authentication or a supported URL format.
- Keep frontend responsibilities separate from backend repository fetching, authentication, and AI review processing.

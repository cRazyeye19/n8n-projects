# n8n Workflow: Gmail to Slack Notification

This document provides a detailed explanation of the n8n workflow defined in [`gmail_to_slack_workflow.json`](gmail_to_slack_workflow.json).

## Workflow Overview

This n8n workflow is designed to automate the process of notifying a Slack channel whenever a new email arrives in a specified Gmail inbox. It extracts key information from the incoming email and posts it as a formatted message in Slack.

### Workflow Structure

The workflow consists of two main nodes:

1.  **Gmail Trigger** (Trigger Node)
2.  **Slack** (Action Node)

These nodes are connected sequentially, meaning the output of the Gmail Trigger node feeds directly into the Slack node.

### Node Details

#### 1. Gmail Trigger

*   **Node Name**: `Gmail Trigger`
*   **Node ID**: `gmailTriggerNode`
*   **Type**: `n8n-nodes-base.gmailTrigger`
*   **Purpose**: This node acts as the starting point of the workflow. It continuously monitors a Gmail account for new emails.
*   **Configuration**:
    *   **Authentication**: `oAuth2` (recommended) - This setting indicates that the node will use OAuth2 credentials to connect to your Gmail account. You will need to set up OAuth2 credentials in your n8n instance for this node to function correctly.
    *   **Event**: `messageReceived` - This specifies that the workflow should be triggered specifically when a new email message is received in the monitored Gmail inbox.

#### 2. Slack

*   **Node Name**: `Slack`
*   **Node ID**: `slackNode`
*   **Type**: `n8n-nodes-base.slack`
*   **Purpose**: This node is responsible for sending messages to a designated Slack channel.
*   **Configuration**:
    *   **Resource**: `message` - Specifies that the node will interact with Slack's messaging capabilities.
    *   **Operation**: `post` - Defines the action to be performed, which is to post a new message.
    *   **Send Message To**: `channel` - Indicates that the message will be sent to a specific Slack channel.
    *   **Channel ID**: `#general` - This is the target Slack channel where the email notifications will be posted. You can change this to any valid channel ID (e.g., `#your-channel-name` or a channel ID like `C12345ABC`).
    *   **Message Text**:
        ```
        New Email Received:
        Sender: {{ $json.from.emailAddress }}
        Subject: {{ $json.subject }}
        Snippet: {{ $json.snippet }}
        ```
        This is the content of the message that will be posted to Slack. It uses n8n expressions (`{{ }}`) to dynamically pull data from the incoming Gmail email:
        *   `$json.from.emailAddress`: Extracts the email address of the sender.
        *   `$json.subject`: Extracts the subject line of the email.
        *   `$json.snippet`: Extracts a short snippet (preview) of the email body.

### Connections

*   The `Gmail Trigger` node is connected to the `Slack` node via the `main` output of the Gmail Trigger and the `main` input of the Slack node. This ensures that every time the Gmail Trigger detects a new email, the data from that email is passed as input to the Slack node, which then processes and sends the notification.

### How it Works

1.  **Trigger**: The `Gmail Trigger` node continuously polls your configured Gmail account.
2.  **Email Detection**: When a new email arrives that matches the trigger criteria (e.g., any new message received), the `Gmail Trigger` node captures its data.
3.  **Data Extraction**: The captured email data, which is available as JSON, is then passed to the `Slack` node.
4.  **Message Formatting**: The `Slack` node uses the expressions defined in its `Message Text` parameter to extract the sender's email address, subject, and a snippet of the body from the incoming email's JSON data.
5.  **Slack Notification**: Finally, the `Slack` node constructs the formatted message and posts it to the specified Slack channel (`#general` in this case).

### Prerequisites and Setup

To use this workflow, you will need:

1.  An active n8n instance (either self-hosted or cloud).
2.  **Gmail Credentials**: You must configure OAuth2 credentials for Gmail within your n8n instance. This involves creating a Google Cloud Project, enabling the Gmail API, and setting up OAuth2 client IDs.
3.  **Slack Credentials**: You must configure credentials for Slack within your n8n instance. This typically involves creating a Slack App and obtaining a Bot User OAuth Token with the necessary permissions (e.g., `chat:write`, `channels:read`).
4.  **Target Slack Channel**: Ensure the Slack channel specified in the `Slack` node (`#general` or your chosen channel) exists and the Slack app has permission to post to it.

### Deployment and Activation

1.  **Import**: Import the `gmail_to_slack_workflow.json` file into your n8n instance.
2.  **Configure Credentials**: Link the Gmail Trigger and Slack nodes to their respective credentials that you have set up in n8n.
3.  **Activate**: Once configured, activate the workflow in n8n. It will then start monitoring your Gmail inbox and sending notifications to Slack automatically.
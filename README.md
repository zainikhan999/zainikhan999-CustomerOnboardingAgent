# 🤖 AutoGen Customer Onboarding Agents

This project creates a multi-agent onboarding experience using [AutoGen](https://github.com/microsoft/autogen), where conversational agents guide users through personal information gathering, preferences setup, and customer engagement.

---

## 🧩 Agents in the System

- **OnboardingPersonalInformationAgent** — Asks for name and location.
- **OnboardingTopicPreferenceAgent** — Collects preferred news topics.
- **CustomerEngagementAgent** — Shares fun stories or jokes based on previous info.
- **CustomerProxyAgent** — Interface for customer responses.

---

## 📦 Installation

```bash
!pip install autogen
```

---

## 🔐 API Key Setup

```python
from google.colab import userdata
deep_seek = userdata.get('DEEP_SEEK_API')
```

---

## ⚙️ OpenRouter & LLM Configuration

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=deep_seek,
)

llm_config = {
    "config_list": [
        {
            "model": "deepseek/deepseek-r1:free",
            "api_key": deep_seek,
            "base_url": "https://openrouter.ai/api/v1",
        }
    ],
    "temperature": 0.7,
}
```

---

## 🤖 Agent Definitions

```python
from autogen import ConversableAgent

onboarding_personal_information_agent = ConversableAgent(
    name="OnboardingPersonalInformationAgent",
    system_message='''You are a helpful customer onboarding agent,
    you are here to help new customers get started with our product.
    Your job is to gather customer's name and location.
    Do not ask for other information. Return 'TERMINATE'
    when you have gathered all the information.''',
    llm_config=llm_config,
    code_execution_config=False,
    human_input_mode="NEVER",
)

onboarding_topic_preference_agent = ConversableAgent(
    name="OnboardingTopicpreferenceAgent",
    system_message='''You are a helpful customer onboarding agent,
    you are here to help new customers get started with our product.
    Your job is to gather customer's preferences on news topics.
    Do not ask for other information.
    Return 'TERMINATE' when you have gathered all the information.''',
    llm_config=llm_config,
    code_execution_config=False,
    human_input_mode="NEVER",
)

customer_engagement_agent = ConversableAgent(
    name="CustomerEngagementAgent",
    system_message='''You are a helpful customer service agent
    here to provide fun for the customer based on the user's
    personal information and topic preferences.
    This could include fun facts, jokes, or interesting stories.
    Make sure to make it engaging and fun!
    Return 'TERMINATE' when you are done.''',
    llm_config=llm_config,
    code_execution_config=False,
    human_input_mode="NEVER",
    is_termination_msg=lambda msg: "terminate" in msg.get("content").lower(),
)

customer_proxy_agent = ConversableAgent(
    name="customer_proxy_agent",
    llm_config=False,
    code_execution_config=False,
    human_input_mode="ALWAYS",
    is_termination_msg=lambda msg: "terminate" in msg.get("content").lower(),
)
```

---

## 💬 Define Conversations

```python
chats = [
    {
        "sender": onboarding_personal_information_agent,
        "recipient": customer_proxy_agent,
        "message":
            "Hello, I'm here to help you get started with our product."
            "Could you tell me your name and location?",
        "summary_method": "reflection_with_llm",
        "summary_args": {
            "summary_prompt" : "Return the customer information "
                             "into as JSON object only: "
                             "{'name': '', 'location': ''}",
        },
        "max_turns": 2,
        "clear_history" : True
    },
    {
        "sender": onboarding_topic_preference_agent,
        "recipient": customer_proxy_agent,
        "message":
                "Great! Could you tell me what topics you are "
                "interested in reading about?",
        "summary_method": "reflection_with_llm",
        "max_turns": 1,
        "clear_history" : False
    },
    {
        "sender": customer_proxy_agent,
        "recipient": customer_engagement_agent,
        "message": "Let's find something fun to read.",
        "max_turns": 1,
        "summary_method": "reflection_with_llm",
    },
]
```

---

## 🚀 Run the Chat

```python
from autogen import initiate_chats

chat_results = initiate_chats(chats)

for chat_result in chat_results:
    print(chat_result.summary)
    print("\n")

for chat_result in chat_results:
    print(chat_result.cost)
    print("\n")
```

---

## 📌 Output

You’ll see printed summaries of each conversation and cost estimations, useful for debugging and insights.

---

## 📝 Notes

- Works best with OpenRouter-compatible models (e.g., DeepSeek, GPT-4, Claude).
- Modify the system messages and summary prompts to fine-tune the onboarding flow.
- `customer_proxy_agent` allows real user input in loop via `human_input_mode="ALWAYS"`.


# Crab AI

[![Crates.io](https://img.shields.io/crates/v/crab_ai.svg)](https://crates.io/crates/crab_ai)
[![Documentation](https://docs.rs/crab_ai/badge.svg)](https://docs.rs/crab_ai)

The Crab AI library provides convenient access to the OpenAI REST API from any Rust application. This library aims to follow the implementation of the official OpenAI SDKs for Python and Node.js as closely as possible. However, it is unofficial and not maintained by OpenAI.

**Note:** This project is still under development, should not be used in production, and many aspects of the structure may change.

## Documentation

The REST API documentation can be found on [platform.openai.com](https://platform.openai.com/docs). The full API of this library can be found in [docs.rs](https://docs.rs/crab_ai).

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
crab_ai = "0.1.5"
```

Then, add this to your crate root:

```rust
extern crate crab_ai;
```

## Usage

The full API of this library can be found in the documentation.

```rust
use crab_ai::resources::chat::{
    ChatCompletionContent::Multiple,
    ChatCompletionContentPart::Image,
    ChatCompletionCreateParams,
    ChatCompletionMessageParam::{System, User},
    ChatModel::Gpt4o,
    Detail, ImageURL,
};
use crab_ai::{ClientOptions, OpenAI};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // OPENAI_API_KEY is required, you can set it in your environment variables.
    // e.g. `export OPENAI_API_KEY="your-api-key"`
    let openai = OpenAI::new(ClientOptions::new())?;

    let completion = openai.chat.completions.create(ChatCompletionCreateParams {
        model: Gpt4o.into(),
        messages: vec![
            System{
                content: "You are a helpful assistant.".to_string(),
                name: None,
            },
            User{
                content: Multiple(vec![Image {
                    image_url: ImageURL {
                        url: "https://inovaveterinaria.com.br/wp-content/uploads/2015/04/gato-sem-raca-INOVA-2048x1365.jpg".to_string(),
                        detail: Some(Detail::Auto),
                    }
                }]),
                name: None,
            },
        ],
        ..Default::default()
    }).await?;

    println!("{:?}", completion);
    Ok(())
}
```

While you can provide an `api_key` directly, we recommend using environment variables to keep your API key secure.

### Examples

Refer to the [examples](https://github.com/decomoraes/crab_ai/tree/main/examples) directory for usage examples.


### OpenAI Assistant Beta

The Crab AI library includes support for the OpenAI Assistant API, which is currently in beta. This feature allows you to create and manage threads, messages, and runs with the Assistant. The Assistant API is designed to help build conversational agents that can interact with users in a more dynamic and context-aware manner.

#### Example: Creating a Thread and a Message

Below is an example demonstrating how to create a thread and a message within that thread using the Assistant API:

```rust
let thread = openai.beta.threads.create(ThreadCreateParams::default()).await?;

let message = openai.beta.threads.messages.create(
    &thread.id,
    MessageCreateParams {
        role: message_create_params::Role::User,
        content: message_create_params::Content::Text("I need to solve the equation `3x + 11 = 14`. Can you help me?".to_string()),
        ..Default::default()
    },
    None,
).await?;
```

This example showcases how to initialize a conversation with a thread and add a message to it.

#### Creating a Run and Polling for Completion

Here's an example of how to create a run using the Assistant API and poll until it reaches a terminal state:

```rust
let run = openai.beta.threads.runs.create_and_poll(
    &thread.id,
    RunCreateParams {
        assistant_id: "asst_ABcDEFgH12345678910xZZZz".to_string(),
        instructions: Some("Please address the user as Jane Doe. The user has a premium account.".to_string()),
        ..Default::default()
    },
    None
).await?;
```

More information on the lifecycle of a Run can be found in the [Run Lifecycle Documentation](https://platform.openai.com/docs/assistants/how-it-works/run-lifecycle).

By integrating these features, the Crab AI library provides a robust interface for utilizing the latest capabilities of the OpenAI API, including the Assistant API currently in beta.
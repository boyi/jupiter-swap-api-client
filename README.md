# jup-swap-api-client

## Introduction

The `jup-swap-api-client` is a Rust client library designed to simplify the integration of the Jupiter Swap API, enabling seamless swaps on the Solana blockchain.

## API Key Required

As of 2025, the Jupiter API requires authentication. Get your API key from [portal.jup.ag](https://portal.jup.ag).

Set the `JUPITER_API_KEY` environment variable:
```bash
export JUPITER_API_KEY="your-api-key"
```

## Getting Started

To use the `jup-swap-api-client` crate in your Rust project, follow these simple steps:

Add the crate to your `Cargo.toml`:

    ```toml
    [dependencies]
    jupiter-swap-api-client = { git = "https://github.com/jup-ag/jupiter-swap-api-client.git", package = "jupiter-swap-api-client"}
    ```

## Examples

Here's a simplified example of how to use the `jup-swap-api-client` in your Rust application:

```rust
use std::env;
use jupiter_swap_api_client::{
    quote::QuoteRequest, swap::SwapRequest, transaction_config::TransactionConfig,
    JupiterSwapApiClient,
};
use solana_sdk::pubkey::Pubkey;

const USDC_MINT: Pubkey = pubkey!("EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v");
const NATIVE_MINT: Pubkey = pubkey!("So11111111111111111111111111111111111111112");
const TEST_WALLET: Pubkey = pubkey!("2AQdpHJ2JpcEgPiATUXjQxA8QmafFegfQwSLWSprPicm");

#[tokio::main]
async fn main() {
    // Get API key from environment
    let api_key = env::var("JUPITER_API_KEY").ok();
    let jupiter_swap_api_client = JupiterSwapApiClient::with_api_key(
        "https://api.jup.ag/swap/v1".to_string(),
        api_key,
    );

    let quote_request = QuoteRequest {
        amount: 1_000_000,
        input_mint: USDC_MINT,
        output_mint: NATIVE_MINT,
        slippage_bps: 50,
        ..QuoteRequest::default()
    };

    // GET /quote
    let quote_response = jupiter_swap_api_client.quote(&quote_request).await.unwrap();
    println!("{quote_response:#?}");

    // POST /swap
    let swap_response = jupiter_swap_api_client
        .swap(&SwapRequest {
            user_public_key: TEST_WALLET,
            quote_response: quote_response.clone(),
            config: TransactionConfig::default(),
        })
        .await
        .unwrap();

    println!("Raw tx len: {}", swap_response.swap_transaction.len());

    // Perform further actions as needed...

    // POST /swap-instructions
    let swap_instructions = jupiter_swap_api_client
        .swap_instructions(&SwapRequest {
            user_public_key: TEST_WALLET,
            quote_response,
            config: TransactionConfig::default(),
        })
        .await
        .unwrap();
    println!("{swap_instructions:#?}");
}

```
For the full example, please refer to the [examples](./example/) directory in this repository.

### Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `JUPITER_API_KEY` | Yes | - | API key from portal.jup.ag |
| `API_BASE_URL` | No | `https://api.jup.ag/swap/v1` | Custom API URL for self-hosted APIs |

### Using Self-hosted APIs

You can set custom URLs via environment variables for any self-hosted Jupiter APIs:

```bash
export API_BASE_URL=https://your-hosted-api.example.com
export JUPITER_API_KEY=your-api-key
```

## Additional Resources

- [Jupiter Swap API Documentation](https://station.jup.ag/docs/v6/swap-api): Learn more about the Jupiter Swap API and its capabilities.
- [jup.ag Website](https://jup.ag/): Explore the official website for additional information and resources.

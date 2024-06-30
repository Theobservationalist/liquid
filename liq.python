import json
import requests
from solana.rpc.api import Client
from solana.publickey import PublicKey
from solana.rpc.types import TokenAccountOpts
from telegram import Bot, Update
from telegram.ext import CallbackContext, CommandHandler, Updater

# Replace with your Telegram bot token
TELEGRAM_BOT_TOKEN = '6741360913:AAEifr63mnIqLWnDIx8piuE2UoLlotTL16k'
SOLANA_RPC_URL = 'https://api.mainnet-beta.solana.com'  # or your preferred RPC endpoint

# Set up the Solana client
client = Client(SOLANA_RPC_URL)

# Function to get token accounts by owner
def get_token_accounts_by_owner(owner_pubkey):
    opts = TokenAccountOpts(program_id=PublicKey("TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA"))
    return client.get_token_accounts_by_owner(owner_pubkey, opts)

# Function to fetch token data from Solscan
def get_token_data(token_address):
    try:
        response = requests.get(f"https://public-api.solscan.io/token?tokenAddress={token_address}")
        if response.status_code == 200:
            return response.json()
        else:
            return None
    except Exception as e:
        print(f"Error fetching token data: {e}")
        return None

# Function to get recent transactions for a token
def get_recent_transactions(token_address, limit=10):
    try:
        response = requests.get(f"https://public-api.solscan.io/token/transactions?tokenAddress={token_address}&limit={limit}")
        if response.status_code == 200:
            return response.json()['data']
        else:
            return []
    except Exception as e:
        print(f"Error fetching recent transactions: {e}")
        return []

# Function to calculate token liquidity
def calculate_liquidity(token_data):
    try:
        total_supply = int(token_data['totalSupply'])
        circulating_supply = int(token_data['circulatingSupply'])
        return total_supply / circulating_supply
    except Exception as e:
        print(f"Error calculating liquidity: {e}")
        return 0

# Function to identify potentially interesting tokens
def find_interesting_tokens(transactions, token_data):
    interesting_tokens = []
    for tx in transactions:
        try:
            # Calculate the transaction amount
            amount = int(tx['amount'], 16) / (10 ** 6)
            # Check if the transaction amount is high
            if amount >= 10000:
                # Calculate the liquidity of the token
                liquidity = calculate_liquidity(token_data)
                # Check if the liquidity is unusual (e.g., above 100)
                if liquidity >= 100:
                    # Add the token to the list of interesting tokens
                    interesting_tokens.append({
                        "token_address": tx['tokenAddress'],
                        "transaction_amount": amount,
                        "liquidity": liquidity
                    })
        except Exception as e:
            print(f"Error processing transaction: {e}")
    return interesting_tokens

# Function to handle the /scan command
def scan_command(update: Update, context: CallbackContext):
    # Get the list of token addresses to scan
    token_addresses = ["YOUR_TOKEN_ADDRESS_1", "YOUR_TOKEN_ADDRESS_2", "YOUR_TOKEN_ADDRESS_3"]
    # Initialize a list to store interesting tokens
    interesting_tokens = []
    # Iterate over each token address
    for token_address in token_addresses:
        # Get recent transactions for the token
        transactions = get_recent_transactions(token_address)
        # Get token data from Solscan
        token_data = get_token_data(token_address)
        # Find potentially interesting tokens
        interesting_tokens.extend(find_interesting_tokens(transactions, token_data))
    # If interesting tokens were found
    if interesting_tokens:
        # Construct a message to be sent to the user
        message = "**Interesting Tokens Found:**\n\n"
        for token in interesting_tokens:
            message += f"Token Address: {token['token_address']}\n"
            message += f"Transaction Amount: {token['transaction_amount']}\n"
            message += f"Liquidity: {token['liquidity']}\n\n"
        # Send the message to the user
        update.message.reply_text(message)
    else:
        # If no interesting tokens were found, send a message to the user
        update.message.reply_text("No interesting tokens found.")

# Function to handle the /start command
def start(update: Update, context: CallbackContext):
    update.message.reply_text("Welcome! Use /scan to scan for interesting tokens.")

# Main function to set up the bot
def main():
    updater = Updater(TELEGRAM_BOT_TOKEN, use_context=True)
    dp = updater.dispatcher

    dp.add_handler(CommandHandler("start", start))
    dp.add_handler(CommandHandler("scan", scan_command))

    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()

# base432import time
from collections import defaultdict
from web3 import Web3

RPC_URL = "https://mainnet.base.org"

WINDOW_BLOCKS = 20
SENDER_THRESHOLD = 10  # number of unique senders


def main():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))

    if not w3.is_connected():
        raise RuntimeError("Cannot connect to Base RPC")

    print("Connected to Base")
    print("Tracking wallets with many unique senders...\n")

    last_block = w3.eth.block_number

    while True:
        try:
            current_block = w3.eth.block_number

            if current_block >= last_block + WINDOW_BLOCKS:

                from_block = current_block - WINDOW_BLOCKS
                to_block = current_block

                receiver_senders = defaultdict(set)

                for b in range(from_block, to_block + 1):

                    block = w3.eth.get_block(b, full_transactions=True)

                    for tx in block.transactions:

                        if tx["to"] and tx.value > 0:
                            receiver_senders[tx["to"]].add(tx["from"])

                print(f"\nBlocks {from_block} → {to_block}")

                for receiver, senders in receiver_senders.items():
                    if len(senders) >= SENDER_THRESHOLD:
                        print("📥 Multi-Sender Receiver")
                        print("Receiver:", receiver)
                        print("Unique senders:", len(senders))
                        print()

                last_block = current_block

            time.sleep(3)

        except Exception as e:
            print("Error:", e)
            time.sleep(5)


if __name__ == "__main__":
    main()

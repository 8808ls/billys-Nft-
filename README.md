# # billys-nft
Copyright 
# create a network client
>>> from xrpl.clients.json_rpc_client import JsonRpcClient
>>> client = JsonRpcClient("https://s.altnet.rippletest.net:51234/")
# create a wallet on the testnet
>>> from xrpl.wallet import generate_faucet_wallet
>>> test_wallet = generate_faucet_wallet(client)
>>> print(test_wallet)
seed: shA5izLnSNFxNwGMV1ar6WJnnsNbo
public_key: ED3CC1BBD0952A60088E89FA502921895FC81FBD79CAE9109A8FE2D23659AD5D56
private_key: -HIDDEN-
classic_address: rBtXmAdEYcno9LWRnAGfT9qBxCeDvuVRZo
# look up account info
>>> from xrpl.models.requests.account_info import AccountInfo
>>> acct_info = AccountInfo(
...         account="rBtXmAdEYcno9LWRnAGfT9qBxCeDvuVRZo",
...         ledger_index="current",
...         queue=True,
...         strict=True,
...     )
>>> response = client.request(acct_info)
>>> result = response.result
>>> import json
>>> print(json.dumps(result["account_data"], indent=4, sort_keys=True))
{
    "Account": "rBtXmAdEYcno9LWRnAGfT9qBxCeDvuVRZo",
    "Balance": "1000000000",
    "Flags": 0,
    "LedgerEntryType": "AccountRoot",
    "OwnerCount": 0,
    "PreviousTxnID": "73CD4A37537A992270AAC8472F6681F44E400CBDE04EC8983C34B519F56AB107",
    "PreviousTxnLgrSeq": 16233962,
    "Sequence": 16233962,
    "index": "FD66EC588B52712DCE74831DCB08B24157DC3198C29A0116AA64D310A58512D7"
}
from xrpl.models.transactions import Payment
from xrpl.transaction import send_reliable_submission
from xrpl.ledger import get_latest_validated_ledger_sequence

current_validated_ledger = get_latest_validated_ledger_sequence(client)

# prepare the transaction
# the amount is expressed in drops, not XRP
# see https://xrpl.org/basic-data-types.html#specifying-currency-amounts
my_tx_payment = Payment(
    account="rMPUKmzmDWEX1tQhzQ8oGFNfAEhnWNFwz",
    amount="2200000",
    destination="rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
    last_ledger_sequence=current_validated_ledger + 20,
    sequence=test_wallet.sequence,
    fee="10",
)
# sign the transaction
my_tx_payment_signed = safe_sign_transaction(my_tx_payment,test_wallet)

# submit the transaction
tx_response = send_reliable_submission(my_tx_payment_signed, client)
>>> from xrpl.models.transactions import Payment
>>> from xrpl.transaction import send_reliable_submission
>>> # prepare the transaction
>>> # the amount is expressed in drops, not XRP
>>> # see https://xrpl.org/basic-data-types.html#specifying-currency-amounts
>>> my_tx_payment = Payment(
>>>     account="rMPUKmzmDWEX1tQhzQ8oGFNfAEhnWNFwz",
>>>     amount="2200000",
>>>     destination="rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe",
>>>     sequence=test_wallet.sequence,
>>> )
>>> # sign the transaction
>>> my_tx_payment_signed = safe_sign_transaction(my_tx_payment, test_wallet)
>>> # submit the transaction
>>> tx_response = send_reliable_submission(my_tx_payment_signed, client)
>>> my_tx_payment
Payment(
    account='rMPUKmzmDWEX1tQhzQ8oGFNfAEhnWNFwz',
    transaction_type=<TransactionType.PAYMENT: 'Payment'>,
    fee=10000,
    sequence=16034065,
    account_txn_id=None,
    flags=0,
    last_ledger_sequence=10268600,
    memos=None,
    signers=None,
    source_tag=None,
    signing_pub_key=None,
    txn_signature=None,
    amount='2200000',
    destination='rPT1Sjq2YGrBMTttX4GZHjKu9dyfzbpAYe',
    destination_tag=None,
    invoice_id=None,
    paths=None,
    send_max=None,
    deliver_min=None
)
"""Interface for all sync network clients to follow."""
from __future__ import annotations

import asyncio

from xrpl.asyncio.clients.client import Client
from xrpl.models.requests.request import Request
from xrpl.models.response import Response


class SyncClient(Client):
    """Interface for all sync network clients to follow."""

    def request(self: SyncClient, request: Request) -> Response:
        """
        Makes a request with a SyncClient.
        Arguments:
            request: The Request to send.
        Returns:
            The Response for the given Request.
        """
        return asyncio.run(self.request_impl(request))
"""High-level ledger methods with the XRPL ledger."""

import asyncio

from xrpl.asyncio.ledger import main
from xrpl.clients.sync_client import SyncClient


def get_latest_validated_ledger_sequence(client: SyncClient) -> int:
    """
    Returns the sequence number of the latest validated ledger.
    Args:
        client: The network client to use to send the request.
    Returns:
        The sequence number of the latest validated ledger.
    Raises:
        XRPLRequestFailureException: if the rippled API call fails.
    """
    return asyncio.run(main.get_latest_validated_ledger_sequence(client))


def get_latest_open_ledger_sequence(client: SyncClient) -> int:
    """
    Returns the sequence number of the latest open ledger.
    Args:
        client: The network client to use to send the request.
    Returns:
        The sequence number of the latest open ledger.
    Raises:
        XRPLRequestFailureException: if the rippled API call fails.
    """
    return asyncio.run(main.get_latest_open_ledger_sequence(client))


def get_fee(client: SyncClient) -> str:
    """
    Query the ledger for the current minimum transaction fee.
    Args:
        client: the network client used to make network calls.
    Returns:
        The minimum fee for transactions.
    Raises:
        XRPLRequestFailureException: if the rippled API call fails.
    """
    return asyncio.run(main.get_fee(client))
-

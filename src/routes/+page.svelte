<script lang="ts">
  import {
    BaseAccount,
    ChainRestAuthApi,
    ChainRestTendermintApi,
    createTransaction,
    createTxRawEIP712,
    createWeb3Extension,
    getEip712TypedData,
    getEip712TypedDataV2,
    getEthereumAddress,
    hexToBase64,
    MsgSend,
    recoverTypedSignaturePubKey,
    SIGN_AMINO,
    TxRestApi,
  } from "@injectivelabs/sdk-ts";
  import { ChainId, EvmChainId } from "@injectivelabs/ts-types";
  import {
    DEFAULT_BLOCK_TIMEOUT_HEIGHT,
    getDefaultStdFee,
    toBigNumber,
    toChainFormat,
  } from "@injectivelabs/utils";

  const handleSigningV1 = async () => {
    const injectiveAddress = "inj133xeq92ak7p87hntvamgq3047kj8jfqhry92a8";
    const ethereumAddress = getEthereumAddress(injectiveAddress);
    const chainId = ChainId.Mainnet;
    const evmChainId = EvmChainId.Mainnet;
    const restEndpoint =
      "https://sentry.lcd.injective.network"; /* getNetworkEndpoints(Network.Mainnet).rest */
    const amount = {
      denom: "inj",
      amount: toChainFormat(0.01).toFixed(),
    };

    /** Account Details **/
    const chainRestAuthApi = new ChainRestAuthApi(restEndpoint);
    const accountDetailsResponse =
      await chainRestAuthApi.fetchAccount(injectiveAddress);
    const baseAccount = BaseAccount.fromRestApi(accountDetailsResponse);
    const accountDetails = baseAccount.toAccountDetails();

    /** Block Details */
    const chainRestTendermintApi = new ChainRestTendermintApi(restEndpoint);
    const latestBlock = await chainRestTendermintApi.fetchLatestBlock();
    const latestHeight = latestBlock.header.height;
    const timeoutHeight = toBigNumber(latestHeight).plus(
      DEFAULT_BLOCK_TIMEOUT_HEIGHT
    );

    /** Preparing the transaction */
    const msg = MsgSend.fromJSON({
      amount,
      srcInjectiveAddress: injectiveAddress,
      dstInjectiveAddress: injectiveAddress,
    });

    /** EIP712 for signing on Ethereum wallets */
    const eip712TypedData = getEip712TypedData({
      msgs: [msg],
      tx: {
        accountNumber: accountDetails.accountNumber.toString(),
        sequence: accountDetails.sequence.toString(),
        timeoutHeight: timeoutHeight.toFixed(),
        chainId: chainId,
      },
      evmChainId,
    });

    /** Use your preferred approach to sign EIP712 TypedData, example with Metamask */
    const signature = await window.ethereum.request({
      method: "eth_signTypedData_v4",
      params: [
        ethereumAddress,
        JSON.stringify(eip712TypedData /* from previous step */),
      ],
    });

    /** Get Public Key of the signer */
    const publicKeyHex = await recoverTypedSignaturePubKey(
      eip712TypedData,
      signature
    );
    const publicKeyBase64 = hexToBase64(publicKeyHex);
    const signatureBuff = Buffer.from(signature.replace("0x", ""), "hex");

    const { txRaw } = createTransaction({
      message: [msg],
      memo: "",
      signMode: SIGN_AMINO,
      fee: getDefaultStdFee(),
      pubKey: publicKeyBase64 /* From previous step */,
      sequence: baseAccount.sequence,
      timeoutHeight: timeoutHeight.toNumber(),
      accountNumber: baseAccount.accountNumber,
      chainId: chainId,
    });
    const web3Extension = createWeb3Extension({
      evmChainId,
    });
    const txRawEip712 = createTxRawEIP712(txRaw, web3Extension);

    /** Append Signatures */
    txRawEip712.signatures = [signatureBuff /* From previous step */];

    /** Broadcast the Transaction */

    const txRestApi = new TxRestApi(restEndpoint);

    const response = await txRestApi.broadcast(txRawEip712);

    /**
     * Once we get the txHash, because we use the Sync mode we
     * are not sure that the transaction is included in the block,
     * it can happen that it's still in the mempool so we need to query
     * the chain to see when the transaction will be included
     */

    /** This will poll querying the transaction and await for it's inclusion in the block */
    const hash = await txRestApi.fetchTxPoll(response.txHash);
    console.log(hash);
    console.log(response);
  };

  const handleSigningV1WithMemo = async () => {
    const injectiveAddress = "inj133xeq92ak7p87hntvamgq3047kj8jfqhry92a8";
    const ethereumAddress = getEthereumAddress(injectiveAddress);
    const chainId = ChainId.Mainnet;
    const evmChainId = EvmChainId.Mainnet;
    const restEndpoint =
      "https://sentry.lcd.injective.network"; /* getNetworkEndpoints(Network.Mainnet).rest */
    const amount = {
      denom: "inj",
      amount: toChainFormat(0.01).toFixed(),
    };

    /** Account Details **/
    const chainRestAuthApi = new ChainRestAuthApi(restEndpoint);
    const accountDetailsResponse =
      await chainRestAuthApi.fetchAccount(injectiveAddress);
    const baseAccount = BaseAccount.fromRestApi(accountDetailsResponse);
    const accountDetails = baseAccount.toAccountDetails();

    /** Block Details */
    const chainRestTendermintApi = new ChainRestTendermintApi(restEndpoint);
    const latestBlock = await chainRestTendermintApi.fetchLatestBlock();
    const latestHeight = latestBlock.header.height;
    const timeoutHeight = toBigNumber(latestHeight).plus(
      DEFAULT_BLOCK_TIMEOUT_HEIGHT
    );

    /** Preparing the transaction */
    const msg = MsgSend.fromJSON({
      amount,
      srcInjectiveAddress: injectiveAddress,
      dstInjectiveAddress: injectiveAddress,
    });

    /** EIP712 for signing on Ethereum wallets */
    const eip712TypedData = getEip712TypedData({
      msgs: [msg],
      tx: {
        accountNumber: accountDetails.accountNumber.toString(),
        sequence: accountDetails.sequence.toString(),
        timeoutHeight: timeoutHeight.toFixed(),
        chainId: chainId,
      },
      evmChainId,
    });

    /** Use your preferred approach to sign EIP712 TypedData, example with Metamask */
    const signature = await window.ethereum.request({
      method: "eth_signTypedData_v4",
      params: [
        ethereumAddress,
        JSON.stringify(eip712TypedData /* from previous step */),
      ],
    });

    /** Get Public Key of the signer */
    const publicKeyHex = await recoverTypedSignaturePubKey(
      eip712TypedData,
      signature
    );
    const publicKeyBase64 = hexToBase64(publicKeyHex);
    const signatureBuff = Buffer.from(signature.replace("0x", ""), "hex");

    const { txRaw } = createTransaction({
      message: [msg],
      memo: "Test",
      signMode: SIGN_AMINO,
      fee: getDefaultStdFee(),
      pubKey: publicKeyBase64 /* From previous step */,
      sequence: baseAccount.sequence,
      timeoutHeight: timeoutHeight.toNumber(),
      accountNumber: baseAccount.accountNumber,
      chainId: chainId,
    });
    const web3Extension = createWeb3Extension({
      evmChainId,
    });
    const txRawEip712 = createTxRawEIP712(txRaw, web3Extension);

    /** Append Signatures */
    txRawEip712.signatures = [signatureBuff /* From previous step */];

    /** Broadcast the Transaction */

    const txRestApi = new TxRestApi(restEndpoint);

    const response = await txRestApi.broadcast(txRawEip712);

    /**
     * Once we get the txHash, because we use the Sync mode we
     * are not sure that the transaction is included in the block,
     * it can happen that it's still in the mempool so we need to query
     * the chain to see when the transaction will be included
     */

    /** This will poll querying the transaction and await for it's inclusion in the block */
    const hash = await txRestApi.fetchTxPoll(response.txHash);
    console.log(hash);
    console.log(response);
  };

  const handleSigningV2 = async () => {
    const injectiveAddress = "inj133xeq92ak7p87hntvamgq3047kj8jfqhry92a8";
    const ethereumAddress = getEthereumAddress(injectiveAddress);
    const chainId = ChainId.Mainnet;
    const evmChainId = EvmChainId.Mainnet;
    const restEndpoint =
      "https://sentry.lcd.injective.network"; /* getNetworkEndpoints(Network.Mainnet).rest */
    const amount = {
      denom: "inj",
      amount: toChainFormat(0.01).toFixed(),
    };

    /** Account Details **/
    const chainRestAuthApi = new ChainRestAuthApi(restEndpoint);
    const accountDetailsResponse =
      await chainRestAuthApi.fetchAccount(injectiveAddress);
    const baseAccount = BaseAccount.fromRestApi(accountDetailsResponse);
    const accountDetails = baseAccount.toAccountDetails();

    /** Block Details */
    const chainRestTendermintApi = new ChainRestTendermintApi(restEndpoint);
    const latestBlock = await chainRestTendermintApi.fetchLatestBlock();
    const latestHeight = latestBlock.header.height;
    const timeoutHeight = toBigNumber(latestHeight).plus(
      DEFAULT_BLOCK_TIMEOUT_HEIGHT
    );

    /** Preparing the transaction */
    const msg = MsgSend.fromJSON({
      amount,
      srcInjectiveAddress: injectiveAddress,
      dstInjectiveAddress: injectiveAddress,
    });

    /** EIP712 for signing on Ethereum wallets */
    const eip712TypedData = getEip712TypedDataV2({
      msgs: [msg],
      tx: {
        accountNumber: accountDetails.accountNumber.toString(),
        sequence: accountDetails.sequence.toString(),
        timeoutHeight: timeoutHeight.toFixed(),
        chainId: chainId,
      },
      evmChainId,
    });

    /** Use your preferred approach to sign EIP712 TypedData, example with Metamask */
    const signature = await window.ethereum.request({
      method: "eth_signTypedData_v4",
      params: [
        ethereumAddress,
        JSON.stringify(eip712TypedData /* from previous step */),
      ],
    });

    /** Get Public Key of the signer */
    const publicKeyHex = await recoverTypedSignaturePubKey(
      eip712TypedData,
      signature
    );
    const publicKeyBase64 = hexToBase64(publicKeyHex);
    const signatureBuff = Buffer.from(signature.replace("0x", ""), "hex");

    const { txRaw } = createTransaction({
      message: [msg],
      memo: "",
      signMode: SIGN_AMINO,
      fee: getDefaultStdFee(),
      pubKey: publicKeyBase64 /* From previous step */,
      sequence: baseAccount.sequence,
      timeoutHeight: timeoutHeight.toNumber(),
      accountNumber: baseAccount.accountNumber,
      chainId: chainId,
    });
    const web3Extension = createWeb3Extension({
      evmChainId,
    });
    const txRawEip712 = createTxRawEIP712(txRaw, web3Extension);

    /** Append Signatures */
    txRawEip712.signatures = [signatureBuff /* From previous step */];

    /** Broadcast the Transaction */

    const txRestApi = new TxRestApi(restEndpoint);

    const response = await txRestApi.broadcast(txRawEip712);

    /**
     * Once we get the txHash, because we use the Sync mode we
     * are not sure that the transaction is included in the block,
     * it can happen that it's still in the mempool so we need to query
     * the chain to see when the transaction will be included
     */

    /** This will poll querying the transaction and await for it's inclusion in the block */
    const hash = await txRestApi.fetchTxPoll(response.txHash);
    console.log(hash);
    console.log(response);
  };
  const handleSigningV2WithMemo = async () => {
    const injectiveAddress = "inj133xeq92ak7p87hntvamgq3047kj8jfqhry92a8";
    const ethereumAddress = getEthereumAddress(injectiveAddress);
    const chainId = ChainId.Mainnet;
    const evmChainId = EvmChainId.Mainnet;
    const restEndpoint =
      "https://sentry.lcd.injective.network"; /* getNetworkEndpoints(Network.Mainnet).rest */
    const amount = {
      denom: "inj",
      amount: toChainFormat(0.01).toFixed(),
    };

    /** Account Details **/
    const chainRestAuthApi = new ChainRestAuthApi(restEndpoint);
    const accountDetailsResponse =
      await chainRestAuthApi.fetchAccount(injectiveAddress);
    const baseAccount = BaseAccount.fromRestApi(accountDetailsResponse);
    const accountDetails = baseAccount.toAccountDetails();

    /** Block Details */
    const chainRestTendermintApi = new ChainRestTendermintApi(restEndpoint);
    const latestBlock = await chainRestTendermintApi.fetchLatestBlock();
    const latestHeight = latestBlock.header.height;
    const timeoutHeight = toBigNumber(latestHeight).plus(
      DEFAULT_BLOCK_TIMEOUT_HEIGHT
    );

    /** Preparing the transaction */
    const msg = MsgSend.fromJSON({
      amount,
      srcInjectiveAddress: injectiveAddress,
      dstInjectiveAddress: injectiveAddress,
    });

    /** EIP712 for signing on Ethereum wallets */
    const eip712TypedData = getEip712TypedDataV2({
      msgs: [msg],
      tx: {
        accountNumber: accountDetails.accountNumber.toString(),
        sequence: accountDetails.sequence.toString(),
        timeoutHeight: timeoutHeight.toFixed(),
        chainId: chainId,
      },
      evmChainId,
    });

    /** Use your preferred approach to sign EIP712 TypedData, example with Metamask */
    const signature = await window.ethereum.request({
      method: "eth_signTypedData_v4",
      params: [
        ethereumAddress,
        JSON.stringify(eip712TypedData /* from previous step */),
      ],
    });

    /** Get Public Key of the signer */
    const publicKeyHex = await recoverTypedSignaturePubKey(
      eip712TypedData,
      signature
    );
    const publicKeyBase64 = hexToBase64(publicKeyHex);
    const signatureBuff = Buffer.from(signature.replace("0x", ""), "hex");

    const { txRaw } = createTransaction({
      message: [msg],
      memo: "Test",
      signMode: SIGN_AMINO,
      fee: getDefaultStdFee(),
      pubKey: publicKeyBase64 /* From previous step */,
      sequence: baseAccount.sequence,
      timeoutHeight: timeoutHeight.toNumber(),
      accountNumber: baseAccount.accountNumber,
      chainId: chainId,
    });
    const web3Extension = createWeb3Extension({
      evmChainId,
    });
    const txRawEip712 = createTxRawEIP712(txRaw, web3Extension);

    /** Append Signatures */
    txRawEip712.signatures = [signatureBuff /* From previous step */];

    /** Broadcast the Transaction */

    const txRestApi = new TxRestApi(restEndpoint);

    const response = await txRestApi.broadcast(txRawEip712);

    /**
     * Once we get the txHash, because we use the Sync mode we
     * are not sure that the transaction is included in the block,
     * it can happen that it's still in the mempool so we need to query
     * the chain to see when the transaction will be included
     */

    /** This will poll querying the transaction and await for it's inclusion in the block */
    const hash = await txRestApi.fetchTxPoll(response.txHash);
    console.log(hash);
    console.log(response);
  };
</script>

<button onclick={handleSigningV1}
  >Sign Transaction V1 Without Memo (Working)</button
>
<button onclick={handleSigningV1WithMemo}
  >Sign Transaction V1 With Memo (Not Working)</button
>
<button onclick={handleSigningV2}
  >Sign Transaction V2 Without Memo (Not Working)</button
>
<button onclick={handleSigningV2WithMemo}
  >Sign Transaction V2 With Memo (Not Working)</button
>

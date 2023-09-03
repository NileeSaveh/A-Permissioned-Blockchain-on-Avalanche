# A-Permissioned-Blockchain-on-Avalanche Network
Currently, preserving critical data is a vital challenge. To address this challenge, we can use a permissioned blockchain-based model on the Avalanche network to protect sensetive data in an immutable environment accessible only to authorized participants.
To create this permissioned blockchain, we employ the Avalanche Subnet-EVM and a custom genesis file. Then, we record data within a single transaction on a permissioned block.
const Web3 = require("web3");

// Set up user custom data
const data = process.argv[2] ? process.argv[2] : "Hello";
const gas = process.argv[3] ? process.argv[3] : 22050;
const blockchainID = process.argv[4]
  ? process.argv[4]
  : "2L7ka8fDcg4JwjgdyhDeui2YgnZjHcUxRij5Agyv2eFpDvKr75";

// Connect to an Ethereum node
const web3 = new Web3(`http://127.0.0.1:9650/ext/bc/${blockchainID}/rpc`);

// Set up two wallets
const walletA = {
  address: "0x8db97C7cEcE249c2b98bDC0226Cc4C2A57BF52FC",
  privateKey:
    "0x56289e99c94b6912bfc12adc093c9b51124f0dc54ac7a766b2bc5ccf558d8027",
};

const walletB = {
  address: "0x0147c562b5aA26eF6e604Ebc1696Ab3Af5876826",
  privateKey:
    "0x01e9438e05bbba12602a666304969a970ee9035c73a46e3912e74589ebe12841",
};

var txID = "";

async function sendETH(fromAddress, toAddress, privateKey, amount, data, gas) {
  // Create transaction object
  let transaction = {
    from: fromAddress,
    to: toAddress,
    gas: web3.utils.toHex(gas),
    value: web3.utils.toHex(web3.utils.toWei(amount, "ether")),
    input: web3.utils.toHex(data),
  };

  console.log("---< START of sending transaction >---");
  for (let t in transaction) {
    console.log(`   - ${t} -> ${transaction[t]}`);
  }
  console.log("---< END of sending transaction >---\n");

  // Sign the transaction
  const signedTx = await web3.eth.accounts.signTransaction(
    transaction,
    privateKey
  );

   // Send the transaction
  return await web3.eth.sendSignedTransaction(
    signedTx.rawTransaction,
    function (error, hash) {
      if (!error) {
        txID = {
          message: hash,
          success: true,
        };
      } else {
        txID = {
          message: error,
          success: false,
        };
      }
    }
  );
}

async function getETH(txID) {
  let blockNo = 0;
  console.log("---< START of getting transaction >---");
  await web3.eth
    .getBlockNumber()
    .then((data) => {
      blockNo = data;
      console.log("<> Block number: " + data);
    })
    .catch((error) => {
      console.log(error);
    });

  await web3.eth
    .getBlock(blockNo)
    .then((data) => {
      console.log("<> Block info: ");
      for (let d in data) {
        console.log(`   - ${d} -> ${data[d]}`);
      }
    })
    .catch((error) => {
      console.log(error);
    });

  await web3.eth
    .getTransaction(txID)
    .then((data) => {
      console.log("<> Transaction info: ");
      for (let d in data) {
        console.log(`   - ${d} -> ${data[d]}`);
      }
      console.log(`   - userInput -> ${web3.utils.hexToUtf8(data["input"])}`);
    })
    .catch((error) => {
      console.log(error);
    });

  console.log("---< END of getting transaction >---");
}

async function main() {
  console.log("\t---------------------");
  console.log("\t|       START       |");
  console.log("\t---------------------");
  await sendETH(
    walletA.address,
    walletB.address,
    walletA.privateKey,
    "0.005",
    data,
    gas
  );

  if (txID.success) {
    console.log("---< START of transaction hash >---");
    console.log(`   ${txID.message}`);
    console.log("---< END of transaction hash >---\n");
  } else {
    console.log("Error: ", txID.message);
  }

  await getETH(txID.message);
  console.log("\t----------------------------");
  console.log("\t| CREDIT by Niloufar Saveh |");
  console.log("\t----------------------------");
}

main();

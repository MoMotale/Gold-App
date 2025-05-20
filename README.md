# Gold-App
Gold Trade application for the international trade of Gold. Providing verifiable gold purchasers and gold suppliers.
// Smart contract pseudocode / Solidity-like logic

const LBMA_Discount = 0.12; // 12% gross discount
const LBMA_Net = 0.09; // 9% net discount

// Main Gold Trade Contract
contract GoldTrade {
    address public buyer;
    address public seller;
    address public vault;
    address public bank;
    uint public goldVolumeKg;
    uint public LBMAPriceUSD;
    bool public SBLC_Issued = false;
    bool public deliveryConfirmed = false;
    uint public totalPaymentDue;
    uint public paymentLocked;
    
    constructor(address _buyer, address _seller, address _vault, address _bank, uint _goldVolumeKg, uint _LBMAPriceUSD) {
        buyer = _buyer;
        seller = _seller;
        vault = _vault;
        bank = _bank;
        goldVolumeKg = _goldVolumeKg;
        LBMAPriceUSD = _LBMAPriceUSD;
        totalPaymentDue = goldVolumeKg * LBMAPriceUSD * (1 - LBMA_Discount);
    }

    function issueSBLC() public {
        require(msg.sender == bank, "Only bank can issue SBLC.");
        SBLC_Issued = true;
    }

    function confirmDelivery() public {
        require(msg.sender == vault, "Only vault can confirm delivery.");
        deliveryConfirmed = true;
    }

    function lockFunds() public payable {
        require(msg.sender == buyer, "Only buyer can lock funds.");
        require(SBLC_Issued, "SBLC must be issued first.");
        require(msg.value == totalPaymentDue, "Incorrect payment amount.");
        paymentLocked = msg.value;
    }

    function releaseFundsToSeller() public {
        require(deliveryConfirmed, "Gold delivery not confirmed.");
        payable(seller).transfer(paymentLocked);
        paymentLocked = 0;
    }

    function refundBuyer() public {
        require(!deliveryConfirmed, "Delivery was confirmed, cannot refund.");
        require(paymentLocked > 0, "No funds to refund.");
        payable(buyer).transfer(paymentLocked);
        paymentLocked = 0;
    }
}

// -------------------
// React Frontend Template (Using ethers.js + TailwindCSS)
// -------------------

// App.jsx
import { useState } from 'react';
import { ethers } from 'ethers';
import './App.css';

const CONTRACT_ADDRESS = "0xYourContractAddress";
const ABI = [
  "function issueSBLC() public",
  "function confirmDelivery() public",
  "function lockFunds() public payable",
  "function releaseFundsToSeller() public",
  "function refundBuyer() public",
  "function totalPaymentDue() public view returns (uint)"
];

function App() {
  const [provider, setProvider] = useState(null);
  const [signer, setSigner] = useState(null);
  const [contract, setContract] = useState(null);
  const [amount, setAmount] = useState('');

  const connectWallet = async () => {
    const prov = new ethers.BrowserProvider(window.ethereum);
    const signer = await prov.getSigner();
    const contract = new ethers.Contract(CONTRACT_ADDRESS, ABI, signer);
    setProvider(prov);
    setSigner(signer);
    setContract(contract);
  };

  const lockFunds = async () => {
    const tx = await contract.lockFunds({ value: ethers.parseEther(amount) });
    await tx.wait();
    alert('Funds Locked');
  };

  const issueSBLC = async () => {
    const tx = await contract.issueSBLC();
    await tx.wait();
    alert('SBLC Issued');
  };

  const confirmDelivery = async () => {
    const tx = await contract.confirmDelivery();
    await tx.wait();
    alert('Delivery Confirmed');
  };

  const releaseFunds = async () => {
    const tx = await contract.releaseFundsToSeller();
    await tx.wait();
    alert('Funds Released');
  };

  const refund = async () => {
    const tx = await contract.refundBuyer();
    await tx.wait();
    alert('Buyer Refunded');
  };

  return (
    <div className="min-h-screen bg-gray-100 p-4">
      <h1 className="text-2xl font-bold mb-4">Gold Trade Smart Contract</h1>
      <button className="bg-blue-600 text-white px-4 py-2 rounded mb-4" onClick={connectWallet}>
        Connect Wallet
      </button>

      <div className="mb-4">
        <input
          type="text"
          placeholder="Amount in ETH"
          value={amount}
          onChange={(e) => setAmount(e.target.value)}
          className="p-2 border rounded mr-2"
        />
        <button onClick={lockFunds} className="bg-green-600 text-white px-4 py-2 rounded">
          Lock Funds
        </button>
      </div>

      <div className="space-x-2">
        <button onClick={issueSBLC} className="bg-yellow-600 text-white px-4 py-2 rounded">Issue SBLC</button>
        <button onClick={confirmDelivery} className="bg-purple-600 text-white px-4 py-2 rounded">Confirm Delivery</button>
        <button onClick={releaseFunds} className="bg-blue-700 text-white px-4 py-2 rounded">Release Funds</button>
        <button onClick={refund} className="bg-red-600 text-white px-4 py-2 rounded">Refund Buyer</button>
      </div>
    </div>
  );
}

export default App;

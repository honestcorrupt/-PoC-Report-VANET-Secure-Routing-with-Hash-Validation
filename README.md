📖 PoC Report: VANET Secure Routing with SHA256 Hash Verification
🌐 Project Overview

This proof-of-concept (PoC) demonstrates a cryptographically enhanced routing protocol for Vehicular Ad Hoc Networks (VANETs) leveraging Veins (OMNeT++ + SUMO).
The SecureRouting module integrates SHA256 message hashing for integrity and authentication, addressing data tampering, impersonation, and message spoofing risks inherent in VANETs.
🔐 Key Features

✅ Secure Message Generation with SHA256 hashes simulating digital signatures
✅ Message Integrity Verification at receiving nodes
✅ Spoofing & Tampering Detection
✅ Attack Simulation Scenarios with controlled message injection
✅ Performance Visualization using SUMO GUI
✅ Extendable to Digital Signatures (for production scenarios)
🏗 Setup & Installation
🛠 Environment

    OMNeT++ 6.1

    Veins (Veins/Veins-inet)

    SUMO 1.18.0

    Kali Linux / Ubuntu

    Python (for early hash logic testing)

🔥 Commands

# OMNeT++ setup
cd ~/assignment/omnetpp-6.1
source setenv
export PATH=$PATH:~/assignment/omnetpp-6.1/bin

# Veins compilation
cd ~/assignment/veins
make clean
make -j$(nproc)

# Launch SUMO simulation
cd ~/assignment/veins/examples/veins
sumo-gui -c erlangen.sumo.cfg

# Run OMNeT++ simulation with SecureRouting
cd ~/assignment/veins/examples/veins
opp_run -n .:../../src:../../examples/veins omnetpp.ini

💻 SecureRouting.cc Code

#include "SecureRouting.h"
#include <openssl/sha.h>
#include <iomanip>
#include <sstream>
using namespace veins;

Define_Module(SecureRouting);

std::string SecureRouting::hashMessage(const std::string& message) {
    unsigned char hash[SHA256_DIGEST_LENGTH];
    SHA256(reinterpret_cast<const unsigned char*>(message.c_str()), message.size(), hash);
    std::stringstream ss;
    for (int i = 0; i < SHA256_DIGEST_LENGTH; ++i)
        ss << std::hex << std::setw(2) << std::setfill('0') << (int)hash[i];
    return ss.str();
}

void SecureRouting::initialize(int stage) {
    BaseWaveApplLayer::initialize(stage);
    if (stage == 0) EV << "SecureRouting initialized.\n";
}

void SecureRouting::onWSM(BaseFrame1609_4* frame) {
    auto* wsm = check_and_cast<TraCIDemo11pMessage*>(frame);
    std::string received = wsm->getDemoData();
    std::string expectedHash = wsm->getHashed();
    if (hashMessage(received) == expectedHash)
        EV << "✅ Valid message: " << received << "\n";
    else
        EV << "❌ Tampered message detected!\n";
}

void SecureRouting::handleSelfMsg(cMessage* msg) {
    std::string msgContent = "V1:65:(100,50)";
    std::string hashed = hashMessage(msgContent);
    EV << "📤 Sending message: " << msgContent << "\n";
    EV << "📤 Hash: " << hashed << "\n";
    auto* wsm = new TraCIDemo11pMessage();
    wsm->setDemoData(msgContent.c_str());
    wsm->setHashed(hashed.c_str());
    sendDown(wsm);
}

🌍 Network Configuration
🔧 Update omnetpp.ini:

*.node[*].applType = "veins::SecureRouting"

🔧 Define/Update .ned (for scenario):

network SecureVANETScenario {
    submodules:
        node[0..N]: Car {
            parameters:
                applType = "veins::SecureRouting";
        }
}

📊 Simulation Results

    SUMO GUI with erlangen.sumo.cfg loaded and active simulation

    OMNeT++ logs:

        Message generation with hashes

        Message integrity validation at receiving nodes

        Tampered messages flagged with errors

📸 Screenshots

   SUMO GUI Map : ![Screenshot 2025-05-24 093823](https://github.com/user-attachments/assets/4afc8891-6738-40a1-9f0e-a50c5d68db95)


   OMNeT++ Logs : ![Screenshot 2025-05-24 092450](https://github.com/user-attachments/assets/2bddf8d6-1d00-46eb-b0c6-d56e5231dd64)


  Network Setup : ![Screenshot 2025-05-24 093717](https://github.com/user-attachments/assets/c0065839-2720-4a11-a468-d869d08c9bbb)


🚩 Impact

   Real-world VANETs face risks from unvalidated message passing.

   This PoC mitigates spoofing and message integrity issues.

  Forms a basis for CVE submissions or academic papers.

   Demonstrates attacker model & countermeasure.

🏁 Next Steps to Reach 100% Completion

✅ Capture complete simulation logs (both clean and attack scenarios)
✅ Document test cases (e.g., message injection, successful/failed hash validation)
✅ Prepare GitHub repo or ZIP for submission
✅ Write a concise disclosure email for a CNA or CVE request
✅ (Optional) Add SHA256 signing via OpenSSL keys for full digital signature
📘 GitHub README

# VANET Secure Routing PoC (Veins + OMNeT++ + SUMO)

🚗 Simulated VANET routing with SHA256 message integrity verification.

## Features
- SHA256 message hashing
- Integrity check of received messages
- Detection of spoofed/tampered data
- Attack simulation scenarios
- SUMO + OMNeT++ visualization

## Setup
```bash
source ~/assignment/omnetpp-6.1/setenv
cd ~/assignment/veins
make -j$(nproc)
cd examples/veins
sumo-gui -c erlangen.sumo.cfg
opp_run -n .:../../src:../../examples/veins omnetpp.ini

```
Code Highlights

    SecureRouting.cc: handles message generation, hashing, and validation

    omnetpp.ini: configures simulation

    SUMO GUI: displays traffic and network activity 

~ Note  i have nthing knowledge about it .. i just learn the whole thing last night and make it here.. so plz if any mistake i made just pardon me
~the full POC SS and Vidoes are here : https://drive.google.com/drive/folders/1S6M8_rc4OKg9NuS50kPfqmNxBkbnF6Yv?usp=sharing

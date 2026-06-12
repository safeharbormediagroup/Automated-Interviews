# Automated-Interviews
Anti Twilio / 0.003 Cost Free


#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <queue>
#include <map>

using namespace std;

// -------------------------------------------------------------
// Candidate Profile & Session State
// -------------------------------------------------------------
struct Candidate {
    string phoneNumber;
    string name;
    int currentStep = 0;
    bool isBlacklisted = false;
    vector<pair<string, string>> interviewLog; // stores <Question, Answer>
};

// -------------------------------------------------------------
// Android SMS Gateway Interface (httpSMS / SMSGate Endpoint Bridge)
// -------------------------------------------------------------
class SmsGateway {
private:
    string apiKey;
    string gatewayUrl;

public:
    SmsGateway(string key, string url) : apiKey(key), gatewayUrl(url) {}

    bool dispatchSms(const string& to, const string& body) {
        // In a live environment, this block utilizes libcurl to fire an HTTP POST request
        // Content-Type: application/json, x-api-key: apiKey
        // Payload: {"to": to, "content": body}
        
        cout << "[SMS OUTBOUND] To: " << to << " | Message: " << body << "\n";
        return true; 
    }
};

// -------------------------------------------------------------
// Automated Interview Screening Engine
// -------------------------------------------------------------
class InterviewEngine {
private:
    shared_ptr<SmsGateway> telemetryBridge;
    map<string, Candidate> activeSessions;
    vector<string> interviewScript;

public:
    InterviewEngine(shared_ptr<SmsGateway> gateway) : telemetryBridge(gateway) {
        // Define standard screening parameters
        interviewScript.push_back("Thanks for applying! To kick things off, can you tell me about your experience with field operations or manual equipment?");
        interviewScript.push_back("Got it. Do you have reliable transportation to travel across active regional job sites?");
        interviewScript.push_back("Perfect. Last question: What is your general weekly availability for shifts?");
    }

    void registerCandidate(const string& phone, const string& name) {
        Candidate newCandidate;
        newCandidate.phoneNumber = phone;
        newCandidate.name = name;
        newCandidate.currentStep = 0;
        
        activeSessions[phone] = newCandidate;

        // Fire opening outreach text
        string greeting = "Hey " + name + "! This is the automated assistant. " + interviewScript[0];
        telemetryBridge->dispatchSms(phone, greeting);
    }

    void handleInboundReply(const string& phone, const string& messageSnippet) {
        // Verification: Ensure candidate session is active
        if (activeSessions.find(phone) == activeSessions.end()) {
            cout << "[LOG] Ignored text from unregistered number: " << phone << "\n";
            return;
        }

        Candidate& candidate = activeSessions[phone];
        int step = candidate.currentStep;

        // Log the response relative to the question asked
        candidate.interviewLog.push_back({interviewScript[step], messageSnippet});
        cout << "[INTERVIEW LOG] " << candidate.name << " responded: \"" << messageSnippet << "\"\n";

        // Basic algorithmic screening layer (can be swapped for an LLM API call)
        if (step == 1 && (messageSnippet.find("no") != string::npos || messageSnippet.find("don't") != string::npos)) {
            telemetryBridge->dispatchSms(phone, "Thank you for the verification. We require active transport for this role. We will keep your file updated.");
            candidate.isBlacklisted = true;
            return;
        }

        // Increment to next structured metric
        candidate.currentStep++;

        if (candidate.currentStep < interviewScript.size()) {
            // Push next contextual screening question
            telemetryBridge->dispatchSms(phone, interviewScript[candidate.currentStep]);
        } else {
            // Screen cycle finalized successfully
            telemetryBridge->dispatchSms(phone, "Excellent, you've completed the preliminary screen. Our team will review your responses and reach out directly.");
            archiveSession(candidate);
        }
    }

private:
    void archiveSession(const Candidate& candidate) {
        cout << "\n=============================================\n";
        cout << " ARCHIVING FINALIZED INTERVIEW: " << candidate.name << "\n";
        cout << " Status: Ready for Review\n";
        for (const auto& log : candidate.interviewLog) {
            cout << "  Q: " << log.first << "\n  A: " << log.second << "\n";
        }
        cout << "=============================================\n\n";
    }
};

// -------------------------------------------------------------
// Application Driver
// -------------------------------------------------------------
int main() {
    // Instantiate hardware pipeline components
    auto localGateway = make_shared<SmsGateway>("LOCAL_SIM_KEY_88291X", "https://api.httpsms.com/v1/messages/send");
    InterviewEngine recruitmentHub(localGateway);

    cout << "Android Local SMS Engine initialized.\n";
    cout << "Tracking incoming webhook logs via local SIM interface...\n\n";

    // 1. Simulator: Candidate applies, database pushes details to engine to initiate loop
    recruitmentHub.registerCandidate("+13365550144", "Marcus");

    // 2. Simulator: Webhook receives text payload from applicant's phone back to your server
    recruitmentHub.handleInboundReply("+13365550144", "I have 2 years of commercial pressure washing experience.");

    // 3. Simulator: Next step in conversational loop
    recruitmentHub.handleInboundReply("+13365550144", "Yes, I have a truck and valid license.");

    // 4. Simulator: Final verification check
    recruitmentHub.handleInboundReply("+13365550144", "Open availability Monday through Friday mornings.");

    return 0;
}

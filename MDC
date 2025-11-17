// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

contract MediChainToken is ERC20, Ownable {
    uint256 public constant INITIAL_SUPPLY = 1000000000 * 10**18;
    
    constructor() ERC20("MediChain Token", "MDC") Ownable(msg.sender) {
        _mint(msg.sender, INITIAL_SUPPLY);
    }
    
    function mint(address to, uint256 amount) external onlyOwner {
        _mint(to, amount);
    }
}

contract MedicalRecordsRegistry is ReentrancyGuard {
    using Counters for Counters.Counter;
    
    MediChainToken public mdcToken;
    
    struct MedicalRecord {
        uint256 recordId;
        address patient;
        string encryptedDataHash;
        string dataType;
        uint256 timestamp;
        bool isActive;
        uint256 accessCount;
    }
    
    struct AccessGrant {
        address grantedTo;
        uint256 expiryTime;
        bool isActive;
        string purpose;
    }
    
    Counters.Counter private _recordIdCounter;
    
    mapping(uint256 => MedicalRecord) public medicalRecords;
    mapping(address => uint256[]) public patientRecords;
    mapping(uint256 => mapping(address => AccessGrant)) public recordAccess;
    mapping(address => bool) public verifiedProviders;
    
    event RecordCreated(uint256 indexed recordId, address indexed patient, string dataType);
    event AccessGranted(uint256 indexed recordId, address indexed patient, address indexed grantedTo, uint256 expiryTime);
    event AccessRevoked(uint256 indexed recordId, address indexed patient, address indexed revokedFrom);
    event RecordAccessed(uint256 indexed recordId, address indexed accessor);
    
    constructor(address _mdcToken) {
        mdcToken = MediChainToken(_mdcToken);
    }
    
    modifier onlyPatient(uint256 recordId) {
        require(medicalRecords[recordId].patient == msg.sender, "Not record owner");
        _;
    }
    
    modifier onlyVerifiedProvider() {
        require(verifiedProviders[msg.sender], "Not a verified provider");
        _;
    }
    
    function addVerifiedProvider(address provider) external {
        verifiedProviders[provider] = true;
    }
    
    function createRecord(string memory encryptedDataHash, string memory dataType) external onlyVerifiedProvider returns (uint256) {
        _recordIdCounter.increment();
        uint256 newRecordId = _recordIdCounter.current();
        
        medicalRecords[newRecordId] = MedicalRecord({
            recordId: newRecordId,
            patient: msg.sender,
            encryptedDataHash: encryptedDataHash,
            dataType: dataType,
            timestamp: block.timestamp,
            isActive: true,
            accessCount: 0
        });
        
        patientRecords[msg.sender].push(newRecordId);
        emit RecordCreated(newRecordId, msg.sender, dataType);
        return newRecordId;
    }
    
    function grantAccess(uint256 recordId, address grantee, uint256 durationInDays, string memory purpose) external onlyPatient(recordId) {
        require(medicalRecords[recordId].isActive, "Record not active");
        uint256 expiryTime = block.timestamp + (durationInDays * 1 days);
        
        recordAccess[recordId][grantee] = AccessGrant({
            grantedTo: grantee,
            expiryTime: expiryTime,
            isActive: true,
            purpose: purpose
        });
        
        emit AccessGranted(recordId, msg.sender, grantee, expiryTime);
    }
    
    function revokeAccess(uint256 recordId, address grantee) external onlyPatient(recordId) {
        recordAccess[recordId][grantee].isActive = false;
        emit AccessRevoked(recordId, msg.sender, grantee);
    }
    
    function accessRecord(uint256 recordId) external view returns (string memory) {
        MedicalRecord memory record = medicalRecords[recordId];
        require(record.isActive, "Record not active");
        
        AccessGrant memory access = recordAccess[recordId][msg.sender];
        require(access.isActive, "No active access grant");
        require(block.timestamp <= access.expiryTime, "Access expired");
        
        return record.encryptedDataHash;
    }
    
    function logRecordAccess(uint256 recordId) external {
        AccessGrant memory access = recordAccess[recordId][msg.sender];
        require(access.isActive && block.timestamp <= access.expiryTime, "Access denied");
        medicalRecords[recordId].accessCount++;
        emit RecordAccessed(recordId, msg.sender);
    }
    
    function getPatientRecords(address patient) external view returns (uint256[] memory) {
        return patientRecords[patient];
    }
}

contract DataMarketplace is ReentrancyGuard {
    using Counters for Counters.Counter;
    
    MediChainToken public mdcToken;
    MedicalRecordsRegistry public recordsRegistry;
    
    struct DataRequest {
        uint256 requestId;
        address researcher;
        string dataCategory;
        string studyPurpose;
        uint256 rewardPerRecord;
        uint256 requiredRecords;
        uint256 collectedRecords;
        uint256 totalBudget;
        bool isActive;
        uint256 deadline;
    }
    
    struct DataContribution {
        address patient;
        uint256 requestId;
        uint256[] recordIds;
        uint256 reward;
        uint256 timestamp;
    }
    
    Counters.Counter private _requestIdCounter;
    Counters.Counter private _contributionIdCounter;
    
    mapping(uint256 => DataRequest) public dataRequests;
    mapping(uint256 => DataContribution) public contributions;
    mapping(address => uint256[]) public patientContributions;
    mapping(uint256 => mapping(address => bool)) public hasContributed;
    
    event DataRequestCreated(uint256 indexed requestId, address indexed researcher, string dataCategory, uint256 rewardPerRecord);
    event DataContributed(uint256 indexed requestId, address indexed patient, uint256 reward);
    event RewardClaimed(uint256 indexed contributionId, address indexed patient, uint256 amount);
    
    constructor(address _mdcToken, address _recordsRegistry) {
        mdcToken = MediChainToken(_mdcToken);
        recordsRegistry = MedicalRecordsRegistry(_recordsRegistry);
    }
    
    function createDataRequest(string memory dataCategory, string memory studyPurpose, uint256 rewardPerRecord, uint256 requiredRecords, uint256 durationInDays) external nonReentrant {
        uint256 totalBudget = rewardPerRecord * requiredRecords;
        require(mdcToken.transferFrom(msg.sender, address(this), totalBudget), "Token transfer failed");
        
        _requestIdCounter.increment();
        uint256 newRequestId = _requestIdCounter.current();
        
        dataRequests[newRequestId] = DataRequest({
            requestId: newRequestId,
            researcher: msg.sender,
            dataCategory: dataCategory,
            studyPurpose: studyPurpose,
            rewardPerRecord: rewardPerRecord,
            requiredRecords: requiredRecords,
            collectedRecords: 0,
            totalBudget: totalBudget,
            isActive: true,
            deadline: block.timestamp + (durationInDays * 1 days)
        });
        
        emit DataRequestCreated(newRequestId, msg.sender, dataCategory, rewardPerRecord);
    }
    
    function contributeData(uint256 requestId, uint256[] memory recordIds) external nonReentrant {
        DataRequest storage request = dataRequests[requestId];
        require(request.isActive, "Request not active");
        require(block.timestamp <= request.deadline, "Request expired");
        require(!hasContributed[requestId][msg.sender], "Already contributed");
        require(request.collectedRecords < request.requiredRecords, "Request fulfilled");
        
        for (uint256 i = 0; i < recordIds.length; i++) {
            (uint256 recordId, address patient, , , , , ) = recordsRegistry.medicalRecords(recordIds[i]);
            require(recordId > 0, "Record does not exist");
            require(patient == msg.sender, "Not record owner");
        }
        
        _contributionIdCounter.increment();
        uint256 contributionId = _contributionIdCounter.current();
        uint256 reward = request.rewardPerRecord;
        
        contributions[contributionId] = DataContribution({
            patient: msg.sender,
            requestId: requestId,
            recordIds: recordIds,
            reward: reward,
            timestamp: block.timestamp
        });
        
        patientContributions[msg.sender].push(contributionId);
        hasContributed[requestId][msg.sender] = true;
        request.collectedRecords++;
        
        require(mdcToken.transfer(msg.sender, reward), "Reward transfer failed");
        emit DataContributed(requestId, msg.sender, reward);
    }
    
    function getActiveRequests() external view returns (uint256[] memory) {
        uint256 activeCount = 0;
        uint256 totalRequests = _requestIdCounter.current();
        
        for (uint256 i = 1; i <= totalRequests; i++) {
            if (dataRequests[i].isActive && block.timestamp <= dataRequests[i].deadline) {
                activeCount++;
            }
        }
        
        uint256[] memory activeRequests = new uint256[](activeCount);
        uint256 index = 0;
        for (uint256 i = 1; i <= totalRequests; i++) {
            if (dataRequests[i].isActive && block.timestamp <= dataRequests[i].deadline) {
                activeRequests[index] = i;
                index++;
            }
        }
        
        return activeRequests;
    }
    
    function closeDataRequest(uint256 requestId) external nonReentrant {
        DataRequest storage request = dataRequests[requestId];
        require(msg.sender == request.researcher, "Not the researcher");
        require(request.isActive, "Already closed");
        
        request.isActive = false;
        uint256 usedBudget = request.collectedRecords * request.rewardPerRecord;
        uint256 refund = request.totalBudget - usedBudget;
        
        if (refund > 0) {
            require(mdcToken.transfer(request.researcher, refund), "Refund failed");
        }
    }
    
    function getPatientEarnings(address patient) external view returns (uint256) {
        uint256 totalEarnings = 0;
        uint256[] memory contributionIds = patientContributions[patient];
        
        for (uint256 i = 0; i < contributionIds.length; i++) {
            totalEarnings += contributions[contributionIds[i]].reward;
        }
        
        return totalEarnings;
    }
}

contract ClinicalTrialMatching is ReentrancyGuard {
    using Counters for Counters.Counter;
    
    MediChainToken public mdcToken;
    
    struct ClinicalTrial {
        uint256 trialId;
        address sponsor;
        string trialName;
        string condition;
        string[] eligibilityCriteria;
        uint256 compensation;
        uint256 participantsNeeded;
        uint256 participantsEnrolled;
        bool isActive;
        uint256 startDate;
        uint256 endDate;
    }
    
    struct Participation {
        uint256 trialId;
        address participant;
        uint256 enrollmentDate;
        bool isCompleted;
        uint256 compensationPaid;
    }
    
    Counters.Counter private _trialIdCounter;
    
    mapping(uint256 => ClinicalTrial) public clinicalTrials;
    mapping(uint256 => mapping(address => Participation)) public participations;
    mapping(address => uint256[]) public participantTrials;
    
    event TrialCreated(uint256 indexed trialId, address indexed sponsor, string trialName);
    event ParticipantEnrolled(uint256 indexed trialId, address indexed participant);
    event TrialCompleted(uint256 indexed trialId, address indexed participant, uint256 compensation);
    
    constructor(address _mdcToken) {
        mdcToken = MediChainToken(_mdcToken);
    }
    
    function createTrial(string memory trialName, string memory condition, string[] memory eligibilityCriteria, uint256 compensation, uint256 participantsNeeded, uint256 durationInDays) external nonReentrant {
        uint256 totalBudget = compensation * participantsNeeded;
        require(mdcToken.transferFrom(msg.sender, address(this), totalBudget), "Budget transfer failed");
        
        _trialIdCounter.increment();
        uint256 newTrialId = _trialIdCounter.current();
        
        clinicalTrials[newTrialId] = ClinicalTrial({
            trialId: newTrialId,
            sponsor: msg.sender,
            trialName: trialName,
            condition: condition,
            eligibilityCriteria: eligibilityCriteria,
            compensation: compensation,
            participantsNeeded: participantsNeeded,
            participantsEnrolled: 0,
            isActive: true,
            startDate: block.timestamp,
            endDate: block.timestamp + (durationInDays * 1 days)
        });
        
        emit TrialCreated(newTrialId, msg.sender, trialName);
    }
    
    function enrollInTrial(uint256 trialId) external nonReentrant {
        ClinicalTrial storage trial = clinicalTrials[trialId];
        require(trial.isActive, "Trial not active");
        require(trial.participantsEnrolled < trial.participantsNeeded, "Trial full");
        require(participations[trialId][msg.sender].participant == address(0), "Already enrolled");
        
        participations[trialId][msg.sender] = Participation({
            trialId: trialId,
            participant: msg.sender,
            enrollmentDate: block.timestamp,
            isCompleted: false,
            compensationPaid: 0
        });
        
        participantTrials[msg.sender].push(trialId);
        trial.participantsEnrolled++;
        
        emit ParticipantEnrolled(trialId, msg.sender);
    }
    
    function completeTrial(uint256 trialId, address participant) external nonReentrant {
        ClinicalTrial storage trial = clinicalTrials[trialId];
        require(msg.sender == trial.sponsor, "Not trial sponsor");
        
        Participation storage participation = participations[trialId][participant];
        require(participation.participant == participant, "Not enrolled");
        require(!participation.isCompleted, "Already completed");
        
        participation.isCompleted = true;
        participation.compensationPaid = trial.compensation;
        
        require(mdcToken.transfer(participant, trial.compensation), "Compensation transfer failed");
        emit TrialCompleted(trialId, participant, trial.compensation);
    }
    
    function getActiveTrials() external view returns (uint256[] memory) {
        uint256 activeCount = 0;
        uint256 totalTrials = _trialIdCounter.current();
        
        for (uint256 i = 1; i <= totalTrials; i++) {
            if (clinicalTrials[i].isActive && clinicalTrials[i].participantsEnrolled < clinicalTrials[i].participantsNeeded) {
                activeCount++;
            }
        }
        
        uint256[] memory activeTrials = new uint256[](activeCount);
        uint256 index = 0;
        for (uint256 i = 1; i <= totalTrials; i++) {
            if (clinicalTrials[i].isActive && clinicalTrials[i].participantsEnrolled < clinicalTrials[i].participantsNeeded) {
                activeTrials[index] = i;
                index++;
            }
        }
        
        return activeTrials;
    }
}

contract DrugSupplyChain {
    using Counters for Counters.Counter;
    
    struct Drug {
        uint256 drugId;
        string drugName;
        string batchNumber;
        address manufacturer;
        uint256 manufactureDate;
        uint256 expiryDate;
        string[] supplyChain;
        address currentHolder;
        bool isVerified;
        bool isRecalled;
    }
    
    Counters.Counter private _drugIdCounter;
    
    mapping(uint256 => Drug) public drugs;
    mapping(string => uint256) public batchToDrugId;
    mapping(address => bool) public verifiedManufacturers;
    mapping(address => bool) public verifiedDistributors;
    
    event DrugManufactured(uint256 indexed drugId, string drugName, string batchNumber, address manufacturer);
    event DrugTransferred(uint256 indexed drugId, address from, address to, string checkpoint);
    event DrugRecalled(uint256 indexed drugId, string reason);
    
    modifier onlyManufacturer() {
        require(verifiedManufacturers[msg.sender], "Not verified manufacturer");
        _;
    }
    
    modifier onlyDistributor() {
        require(verifiedDistributors[msg.sender], "Not verified distributor");
        _;
    }
    
    function addManufacturer(address manufacturer) external {
        verifiedManufacturers[manufacturer] = true;
    }
    
    function addDistributor(address distributor) external {
        verifiedDistributors[distributor] = true;
    }
    
    function manufactureDrug(string memory drugName, string memory batchNumber, uint256 expiryDate) external onlyManufacturer returns (uint256) {
        require(batchToDrugId[batchNumber] == 0, "Batch already exists");
        
        _drugIdCounter.increment();
        uint256 newDrugId = _drugIdCounter.current();
        
        string[] memory initialChain = new string[](1);
        initialChain[0] = "Manufactured";
        
        drugs[newDrugId] = Drug({
            drugId: newDrugId,
            drugName: drugName,
            batchNumber: batchNumber,
            manufacturer: msg.sender,
            manufactureDate: block.timestamp,
            expiryDate: expiryDate,
            supplyChain: initialChain,
            currentHolder: msg.sender,
            isVerified: true,
            isRecalled: false
        });
        
        batchToDrugId[batchNumber] = newDrugId;
        emit DrugManufactured(newDrugId, drugName, batchNumber, msg.sender);
        return newDrugId;
    }
    
    function transferDrug(uint256 drugId, address to, string memory checkpoint) external {
        Drug storage drug = drugs[drugId];
        require(drug.currentHolder == msg.sender, "Not current holder");
        require(!drug.isRecalled, "Drug is recalled");
        require(block.timestamp < drug.expiryDate, "Drug expired");
        
        drug.supplyChain.push(checkpoint);
        drug.currentHolder = to;
        
        emit DrugTransferred(drugId, msg.sender, to, checkpoint);
    }
    
    function recallDrug(uint256 drugId, string memory reason) external {
        Drug storage drug = drugs[drugId];
        require(drug.manufacturer == msg.sender, "Not manufacturer");
        drug.isRecalled = true;
        emit DrugRecalled(drugId, reason);
    }
    
    function verifyDrugByBatch(string memory batchNumber) external view returns (bool isAuthentic, bool isRecalled, bool isExpired, address manufacturer) {
        uint256 drugId = batchToDrugId[batchNumber];
        
        if (drugId == 0) {
            return (false, false, false, address(0));
        }
        
        Drug memory drug = drugs[drugId];
        return (drug.isVerified, drug.isRecalled, block.timestamp >= drug.expiryDate, drug.manufacturer);
    }
    
    function getSupplyChain(uint256 drugId) external view returns (string[] memory) {
        return drugs[drugId].supplyChain;
    }
}

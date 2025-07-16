# 🏦 PSD2 Fraud Observatory

Enterprise fraud detection system for SEPA payments using Splunk analytics.

## 🎯 Project Overview

Real-time fraud detection system analyzing **5,000 SEPA transactions** across 10 EU countries to identify high-risk payment patterns and ensure PSD2 compliance.

**Key Stats:**
- **Total Volume**: €38.8M analyzed
- **Fraud Rate**: 11.8% (590 cases)
- **Countries**: 10 EU member states
- **Time Period**: 30 days (June-July 2025)

## 🚨 Critical Fraud Findings

### **High-Risk Geographic Corridors**
- **IT → RO: 90.5% fraud rate** (19/21 transactions)
- **FR → PL: 70.8% fraud rate** (17/24 transactions)
- **ES → BG: 70.0% fraud rate** (14/20 transactions)
- **IT → BG: 59.1% fraud rate** (13/22 transactions)

### **Device Risk Analysis**
- **Critical Devices: 62.5% fraud rate** (€1.2M exposure)
- **Medium Devices: 6.2% fraud rate** (€4.9M volume)
- **Low Devices: 6.7% fraud rate** (€1.5M volume)

### **Time-Based Patterns**
- **1 AM: 33.3% fraud rate** - Peak risk period
- **4 AM: 28.6% fraud rate** - High overnight risk
- **6-8 PM: 0% fraud rate** - Safest processing window

### **Amount-Based Risk**
- **€50K+ transactions: 66.7% fraud rate**
- **€10K-50K transactions: 48.6% fraud rate**
- **€0-1K transactions: 8.2% fraud rate** (baseline)

## 🔍 Core Detection Queries

### **Geographic Risk Analysis**
```spl
index=psd2_transactions_a 
| stats count as transactions, 
        count(eval(is_fraud="true")) as fraud_cases,
        sum(amount) as volume
  by sender_country, receiver_country
| eval fraud_rate=round((fraud_cases/transactions)*100,1)
| where fraud_rate > 20 OR volume > 1000000
| sort - fraud_rate
```

### **Device Risk Intelligence**
```spl
index=psd2_transactions_a 
| stats count as transactions,
        count(eval(is_fraud="true")) as fraud_cases,
        avg(amount) as avg_amount
  by device_risk
| eval fraud_rate=round((fraud_cases/transactions)*100,1)
| sort device_risk
```

### **Temporal Risk Patterns**
```spl
index=psd2_transactions_a 
| eval hour=strftime(_time, "%H")
| stats count as transactions,
        count(eval(is_fraud="true")) as fraud_cases
  by hour
| eval fraud_rate=round((fraud_cases/transactions)*100,1)
| sort hour
```

### **Amount-Based Segmentation**
```spl
index=psd2_transactions_a 
| eval amount_tier=case(
    amount <= 1000, "Small (€0-1K)",
    amount <= 10000, "Medium (€1K-10K)", 
    amount <= 50000, "Large (€10K-50K)",
    amount > 50000, "Critical (€50K+)")
| stats count as transactions,
        count(eval(is_fraud="true")) as fraud_cases
  by amount_tier
| eval fraud_rate=round((fraud_cases/transactions)*100,1)
| sort - fraud_rate
```

## 🚨 Real-Time Alerts

### **Critical Geographic Risk**
```spl
index=psd2_transactions_a earliest=-15m
| where sender_country!=receiver_country
| eval risk_corridor=sender_country + "→" + receiver_country
| where risk_corridor IN ("IT→RO", "FR→PL", "ES→BG", "IT→BG")
| eval alert_severity="CRITICAL"
```

### **Device Risk Alert**
```spl
index=psd2_transactions_a earliest=-10m
| where device_risk="Critical" AND amount > 1000
| eval alert_severity="HIGH"
```

### **Large Amount Alert**
```spl
index=psd2_transactions_a earliest=-5m
| where amount > 50000
| eval alert_severity="CRITICAL"
```

## ⚡ Quick Setup

1. **Clone repository**
```bash
git clone https://github.com/Abayommy/psd2-fraud-observatory.git
```

2. **Generate data**
```bash
python3 data/psd2_data_generator_2025.py
```

3. **Create Splunk index**
```bash
splunk add index psd2_transactions_a -maxDataSize 5000
```

4. **Ingest data**
- Upload JSON file to Splunk
- Set index: `psd2_transactions_a`
- Set source type: `_json`

## 📊 Business Impact

- **€1.2M fraud exposure** identified from critical devices
- **90.5% fraud rate** in IT→RO corridor requires immediate action
- **10x higher fraud risk** for critical vs low-risk devices
- **Real-time alerting** prevents fraud completion

## 🔒 PSD2 Compliance

- **Strong Customer Authentication (SCA)** monitoring
- **API performance** tracking
- **Regulatory reporting** capabilities
- **Audit trail** maintenance

## 📁 Repository Structure

```
psd2-fraud-observatory/
├── README.md
├── data/
│   ├── psd2_data_generator_2025.py
│   └── sample_transactions.json
├── screenshots/
│   ├── geographic_risk.png
│   ├── device_analysis.png
│   └── temporal_patterns.png
├── splunk/
│   ├── fraud_detection_queries.spl
│   └── alert_definitions.spl
└── docs/
    └── setup_guide.md
```

## 🛠️ Technologies

- **Splunk Enterprise** 9.0+
- **Python** 3.8+
- **JSON** data format
- **SPL** (Splunk Processing Language)

## 📞 Contact

**Abayomi Ajayi** - Product Manager → Cybersecurity Professional
- GitHub: [@Abayommy](https://github.com/Abayommy)
- LinkedIn: [Connect](https://linkedin.com/in/abayommy)

## 📄 License

MIT License

---

*Built for financial security and regulatory compliance*


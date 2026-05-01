# E20 CO326: Bearing Failure Prediction System

## Group Members

- Team E20 - CO326 Project Group
- E/20/062
- E/20/271
- E/20/367
- E/20/370

## Project Description

This project implements a real-time bearing failure prediction system using IoT sensors and edge AI. The system monitors bearing vibrations, detects anomalies using statistical analysis, and provides real-time alerts through an MQTT-based architecture.

**Key Features:**
- Real-time sensor data collection and processing
- MQTT broker for publish-subscribe messaging
- Edge AI anomaly detection using Z-score statistical analysis
- Node-RED dashboard for real-time visualization and monitoring
- Docker containerization for easy deployment

**Application:** Predictive maintenance for industrial motors and bearing systems to prevent catastrophic failures and reduce downtime.

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     IoT Bearing Monitoring System                 │
└─────────────────────────────────────────────────────────────────┘

Message Broker:
├─ Mosquitto MQTT Broker (Port 1883)
│  └─ Handles all pub/sub messaging

Data Processing & Visualization:
├─ Python MQTT Publisher (Sensor Data Generator)
│  ├─ Simulates realistic bearing vibration data
│  └─ Publishes to: sensors/{GROUP}/bearing/data
│
├─ Python Edge AI (Anomaly Detection)
│  ├─ Subscribes to: sensors/{GROUP}/bearing/data
│  ├─ Real-time anomaly detection (Z-score analysis)
│  └─ Publishes alerts to: alerts/{GROUP}/bearing/status
│
└─ Node-RED Dashboard
   ├─ Real-time data visualization
   ├─ Alert notifications
   └─ Manual control interface (Port 1880)

Data Flow:
Sensor Data → MQTT Broker → [Python AI + Node-RED] → Alerts & Dashboard → Relay Control
```

**Docker Services:**
- **mqtt** (eclipse-mosquitto:2.0): Message broker
- **node-red** (nodered/node-red:latest): Dashboard and visualization
- **python-publisher**: MQTT data publisher with simulated sensor readings
- **python-ai**: Edge AI anomaly detection engine

## How to Run

### Prerequisites
- Docker and Docker Compose installed
- Python 3.8+ (if running outside Docker)
- Required Python packages: `paho-mqtt`, `numpy`

### Step 1: Navigate to Project Directory
```bash
cd c:\Users\ASUS TUF\Desktop\hardware\e20-co326-bearing-failure-prediction
```

### Step 2: Start All Services with Docker Compose
```bash
docker-compose up -d
```

This command will:
- Start the Mosquitto MQTT broker on port 1883
- Start Node-RED dashboard on port 1880
- Build and run the Python MQTT publisher
- Build and run the Python anomaly detection AI

### Step 3: Access the Services

**Node-RED Dashboard:**
- Open browser: `http://localhost:1880`
- Monitor real-time vibration data
- View anomaly alerts
- Control system parameters

**MQTT Broker:**
- Accessible on `localhost:1883` from local services
- Anonymous connections allowed (configured in `mosquitto/mosquitto.conf`)

**Python Services:**
- Automatically running in containers
- Logs visible via: `docker logs python-publisher` and `docker logs python-ai`

### Step 4: Monitor System
```bash
# View all running containers
docker ps

# View logs from specific service
docker logs python-ai -f
docker logs python-publisher -f

# Stop all services
docker-compose down
```

## MQTT Topics Used

### Data Topics

| Topic | Publisher | Subscriber | Payload |
|-------|-----------|-----------|---------|
| `sensors/{GROUP}/bearing/data` | Python Publisher | Python AI, Node-RED | `{"vibration": float, "degradation_level": float, "timestamp": float}` |

### Alert Topics

| Topic | Publisher | Subscriber | Payload |
|-------|-----------|-----------|---------|
| `alerts/{GROUP}/bearing/status` | Python AI | Node-RED, Dashboard | `{"status": string, "vibration": float, "anomaly": bool, "timestamp": float}` |

### MQTT Broker Configuration
- **Listener Port:** 1883
- **Anonymous Access:** Enabled
- **Configuration File:** `mosquitto/mosquitto.conf`

### Message Format Examples

**Sensor Data (Input):**
```json
{
  "vibration": 3.456,
  "degradation_level": 0.125,
  "timestamp": 1714521600.123
}
```

**Alert Message (Output):**
```json
{
  "status": "WARNING - abnormal vibration",
  "vibration": 5.234,
  "anomaly": true,
  "timestamp": 1714521602.456
}
```

## Results (Screenshots)

### Dashboard Visualization
- Node-RED Dashboard displays:
  - Real-time vibration magnitude graph
  - Current degradation level gauge
  - System status indicator (NORMAL/WARNING/CRITICAL)
  - Alert log with timestamps
  - Manual override controls

### Anomaly Detection Results
- **NORMAL:** Vibration values < 5.0 and Z-score < 2.5
- **WARNING:** Vibration 5.0-7.0 or Z-score > 2.5
- **CRITICAL:** Vibration > 7.0 or sustained high values (bearing failure imminent)

### Performance Metrics
- Detection latency: ~100-200ms (Python AI processing)
- Data collection rate: 1 sample every 2 seconds
- Rolling window size: 20 samples (~40 seconds baseline)

## Challenges

### Technical Challenges

1. **Sensor Noise & False Positives**
   - Solution: Implemented rolling window Z-score analysis with 20-sample baseline
   - Added multi-threshold detection (5.0 and 7.0 G acceleration)

2. **MQTT Network Latency**
   - Challenge: Delays in Docker container communication
   - Solution: Optimized message frequency and Docker network configuration

3. **Real-time Processing Requirements**
   - Challenge: Need for sub-second anomaly detection
   - Solution: Edge AI processing on local Python container, not cloud-based

4. **Data Quality Variation**
   - Challenge: Different sensor calibration between hardware and simulation
   - Solution: Normalized vibration data and adaptive thresholds

5. **Docker Dependency Management**
   - Challenge: Service startup order and connectivity
   - Solution: Configured `depends_on` in docker-compose.yml

### Integration Challenges

1. **Node-RED Dashboard Design**
   - Challenge: Visualizing multi-dimensional sensor data effectively
   - Solution: Implemented gauge charts, line graphs, and status indicators

2. **Python Environment Compatibility**
   - Resolved NumPy and Paho-MQTT version conflicts through Dockerfile specifications

## Future Improvements

### Phase 2 Enhancements

1. **Advanced ML Models**
   - Implement LSTM neural networks for time-series anomaly detection
   - Add transfer learning for cross-bearing generalization
   - Develop predictive remaining useful life (RUL) estimation

2. **Multi-Sensor Fusion**
   - Integrate temperature sensors for thermal analysis
   - Add acoustic emission sensors for noise-based diagnostics
   - Combine vibration + thermal + acoustic data

3. **Cloud Integration**
   - Deploy to AWS IoT Core or Azure IoT Hub
   - Historical data storage in cloud database
   - Machine learning model training on accumulated data

4. **Enhanced Dashboard Features**
   - 3D vibration vector visualization
   - Trend analysis and predictive alerts
   - Multi-bearing/motor monitoring on single dashboard
   - Mobile app for remote monitoring

5. **Reliability & Robustness**
   - Implement data logging for offline analysis
   - Add backup MQTT brokers for high availability
   - Develop automatic failover mechanisms
   - Add system health monitoring and auto-recovery

6. **Performance Optimization**
   - Implement data compression for reduced bandwidth
   - Add edge caching and local data storage
   - Optimize Python AI model for faster inference
   - Parallel processing for multi-bearing systems

7. **Standardization & Documentation**
   - RESTful API for external system integration
   - Standard data formats (IEC 60750 bearing monitoring)
   - Comprehensive API documentation
   - Training materials and deployment guides

### Scalability Goals
- Support monitoring of 50+ industrial motors simultaneously
- Reduce detection latency to <100ms
- Implement distributed edge AI at multiple locations
- Create federation of MQTT brokers for large-scale deployment

---

## Additional Resources

- **Node-RED:** https://nodered.org/
- **Mosquitto MQTT:** https://mosquitto.org/
- **Paho Python MQTT:** https://www.eclipse.org/paho/index.php?page=clients/python/index.php
- **Docker Documentation:** https://docs.docker.com/

## License

This project is part of the CO326 Engineering coursework (E20).

---

**Last Updated:** May 2026
**Project Status:** Active Development

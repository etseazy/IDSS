# Custom live IDS prototype (48-feature model)

This prototype loads the existing `ids_v1_pipeline.joblib` and uses the saved 48-column schema. It does **not** retrain the model or use CICFlowMeter.

## Files to keep together

- `live_ids.py`
- `ids_v1_pipeline.joblib`
- `ids_v1_common.py`
- `ids_v1_feature_schema.json`
- `ids_v1_metadata.json`

## Install

Use the same Python environment/version used to train the artifact where possible. The artifact was created with scikit-learn 1.9.1, so install a matching version if available:

```bash
python -m venv .venv
# macOS/Linux:
source .venv/bin/activate
# Windows PowerShell:
# .venv\\Scripts\\Activate.ps1
python -m pip install -r requirements_live_ids.txt
```

Copy the five files above into one directory. On Windows, run capture from an elevated PowerShell and ensure Npcap is installed. On macOS/Linux, packet capture may require `sudo` or capture permissions.

## Replay a PCAP/PCAPNG

```bash
python live_ids.py --pcap ./pac_cap_1.pcapng --csv replay_predictions.csv
```

## Live capture

List interfaces using Scapy (`python -c "from scapy.all import get_if_list; print(get_if_list())"`), then specify one:

```bash
python live_ids.py --live --iface en0 --csv live_predictions.csv
```

Examples: `en0` on macOS; an interface name returned by Scapy on Windows/Linux. Use only on networks you own or have permission to monitor. Stop with Ctrl+C.

## Important limitations (read this)

This is a **prototype** that sends calculated values through the existing model. It emits the exact 48 column names and units intended by the CIC-IDS2017 schema, but it is **not yet proven to reproduce CICFlowMeter's formulas bit-for-bit**. In particular, `min_seg_size_forward`, active/idle segmentation, header-length conventions, direction assignment, and packet-length semantics may differ by extractor/version. Live inference working does not establish detection accuracy. Treat predictions as experimental until the feature formulas are cross-checked against reference rows and a representative labeled PCAP.

The model threshold is read from `ids_v1_metadata.json` (0.65 for the saved artifact). A completed flow is scored after its idle timeout (default 15 seconds) or active timeout (default 120 seconds), so this is flow-completion detection, not instant packet-level alerting.

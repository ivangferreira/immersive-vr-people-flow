# Immersive VR and Computer Vision for Urban Decision-Making

Code and data for the paper:

> Eduardo Henrique dos Santos and Ivan Fellipy Gonçalves Ferreira. **Immersive Virtual Reality Visualization Platform Integrated with IoT and Computer Vision to Support Decision-Making in Urban Environments.** Submitted to the *Journal on Interactive Systems* (JIS), 2026.

The study counts people crossing a virtual line in a camera feed (YOLOv8 detection and ByteTrack tracking), stores only aggregated counters, and presents movement intensity in a Virtual Reality scene built on a 360-degree panorama of the monitored space.

## Contents

| Folder | Contents |
|---|---|
| `people-counting/` | Python pipeline: video capture, YOLOv8 person detection, ByteTrack tracking, virtual-line counting, and periodic snapshots |
| `data/` | `recording_share.json`, the 62-minute collection analyzed in the paper (361 snapshots) |
| `vr-scene/` | A-Frame scene that wraps a 360-degree panorama of the monitored corridor around the viewer |

## People counting

Requires Python 3.9 or newer.

```bash
cd people-counting
pip install -r requirements.txt

# Webcam 0, vertical line through the middle of the frame
python main.py --source 0 --line 50%,0%,50%,100% --camera-id webcam-01

# Video file or RTSP stream, line given in pixels
python main.py --source video.mp4 --line 100,300,1180,300 --camera-id entrance
```

| Option | Meaning |
|---|---|
| `--source` | Video file path, webcam index (for example `0`), or RTSP URL |
| `--line` | Virtual line as `x1,y1,x2,y2`, in pixels or in percentages of the frame |
| `--camera-id` | Identifier written to every snapshot (required) |
| `--model` | YOLO weights (default `yolov8n.pt`, downloaded automatically) |
| `--interval` | Seconds between snapshots (default `10`) |
| `--storage` | Output file in JSON Lines format (default `counts.json`) |
| `--no-show` | Run without the preview window |

A person who crosses the line from the bottom-right side to the top-left side of the direction P1→P2 counts as an entry; the opposite direction counts as an exit. Swap the two points to invert this.

**Privacy.** Frames are processed in memory only. The output stores the camera identifier, a timestamp, and cumulative counters. No images, video, face crops, or tracker identifiers are saved.

## Data

`data/recording_share.json` is in JSON Lines format, one snapshot per line:

```json
{"camera_id": "minha-webcam", "timestamp": "2026-05-21T00:20:21.094134+00:00", "in": 1, "out": 0, "total": 1}
```

- One camera (`minha-webcam`), 21 May 2026, from 00:20:11 to 01:22:10 UTC (about 62 minutes), with a median interval of 10 seconds between snapshots.
- `in`, `out`, and `total` are cumulative since the start of the run. The final snapshot records 31 entries, 23 exits, and 54 passages; the researchers' manual count was 32, 22, and 54.

The count for an interval is the difference between the last and the first snapshot inside it. For example, for minutes 35 to 40 of the collection:

```python
import json

rows = [json.loads(line) for line in open("data/recording_share.json")]
window = [r for r in rows if "2026-05-21T00:55:11" <= r["timestamp"] < "2026-05-21T01:00:12"]
print(window[-1]["in"] - window[0]["in"])  # 11 entries
```

## VR scene

`vr-scene/index.html` is an A-Frame 1.4 scene that maps the panorama `image.png` onto an open cylinder around a first-person camera. Browsers block textures loaded from `file://`, so serve the folder locally:

```bash
cd vr-scene
python -m http.server 8080
# then open http://localhost:8080
```

This is the base panoramic scene. The intensity markers described in the paper and the later Unity version for Meta Quest are not included.

## Not included

- Video or images from the counting camera: the system never stores them.
- The usability questionnaire (SUS) responses, which contain personal data. Anonymized responses are available from the authors upon request.

## How to cite

```bibtex
@unpublished{santos2026immersive,
  author = {Santos, Eduardo Henrique dos and Ferreira, Ivan Fellipy Gon{\c{c}}alves},
  title  = {Immersive Virtual Reality Visualization Platform Integrated with {IoT} and Computer Vision to Support Decision-Making in Urban Environments},
  note   = {Submitted to the Journal on Interactive Systems},
  year   = {2026}
}
```

## Authors

- Eduardo Henrique dos Santos ([ORCID 0009-0009-2902-3728](https://orcid.org/0009-0009-2902-3728))
- Ivan Fellipy Gonçalves Ferreira ([ORCID 0009-0009-3991-9982](https://orcid.org/0009-0009-3991-9982))

Instituto de Tecnologia e Liderança (Inteli), São Paulo, Brazil. Advisor: Andre Godoi.

## License

- Code (`people-counting/` and `vr-scene/index.html`): [MIT](LICENSE).
- Data (`data/`) and the panorama image (`vr-scene/image.png`): [CC BY 4.0](LICENSE-DATA.md).

# Therapy Engagement Analysis Tool

A computer vision tool that analyzes engagement levels between a child and therapist during therapy sessions by tracking gaze direction, facial emotions, and hand movements.

## Features

- **Face Detection & Recognition**: Identifies and distinguishes between child and therapist
- **Gaze Tracking**: Estimates eye gaze direction using facial landmarks
- **Emotion Recognition**: Detects facial emotions (happy, sad, angry, fear, etc.)
- **Hand Gesture Detection**: Tracks hand movements and positions
- **Engagement Scoring**: Calculates real-time engagement scores based on multiple factors
- **Video Processing**: Processes entire videos with progress tracking

## Requirements

### Python Dependencies

Create a `requirements.txt` file with the following content (or use the provided one):
```
opencv-python>=4.5.0
numpy>=1.21.0
dlib>=19.22.0
face-recognition>=1.3.0
mediapipe>=0.8.9
fer>=22.4.0
tqdm>=4.62.0
```

Then install all dependencies at once:
```bash
pip install -r requirements.txt
```

**Alternative manual installation:**
```bash
pip install opencv-python numpy dlib face-recognition mediapipe fer tqdm
```

### Additional Model Files

Download the following required model file:
- **dlib facial landmarks predictor**: `shape_predictor_68_face_landmarks.dat`
  - Download from: [dlib.net](http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2)
  - Extract and place in your project directory

## Installation

1. **Set up your project**
   ```bash
   # Create a new directory for your project
   mkdir therapy-analysis
   cd therapy-analysis
   
   # Save the provided Python script as therapy_analysis.py
   # Save the requirements.txt file (provided above)
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Download the dlib model**
   ```bash
   wget http://dlib.net/files/shape_predictor_68_face_landmarks.dat.bz2
   bunzip2 shape_predictor_68_face_landmarks.dat.bz2
   ```

## Usage

### Basic Usage

1. **Update file paths in the script**:
   ```python
   # Set your input and output video paths
   input_video = "path/to/your/input_video.mp4"
   output_video = "path/to/your/output_video.mp4"
   ```

2. **Run the analysis**:
   ```bash
   python therapy_analysis.py
   ```

### Customizing for Your Data

#### Video Requirements
- **Format**: MP4, AVI, MOV (most common video formats supported)
- **Content**: Should contain exactly 2 people (child and therapist)
- **Quality**: Higher resolution videos will provide better accuracy
- **Lighting**: Well-lit scenes improve face and emotion detection

#### Adjusting Parameters

**Face Detection Sensitivity**:
```python
# In face_recognition.face_locations()
number_of_times_to_upsample=2  # Increase for better detection of smaller faces
```

**Hand Detection Sensitivity**:
```python
# In mp_hands.Hands()
min_detection_confidence=0.1  # Lower = more sensitive (0.1-0.9)
min_tracking_confidence=0.1   # Lower = more sensitive (0.1-0.9)
max_num_hands=4              # Maximum hands to detect
```

**Emotion Detection Confidence**:
```python
# In robust_emotion_classification()
min_confidence=0.3  # Minimum confidence threshold (0.1-0.9)
```

#### Customizing Engagement Calculation

The engagement score is calculated based on:
- **Gaze movement and direction** (weight: 0.1-0.2)
- **Facial emotion** (happy: +0.2, negative emotions: -0.1)
- **Hand movement** (weight: 0.1-0.3)
- **Proximity of interactions** (closer = higher engagement)

Modify the `calculate_engagement()` function to adjust these weights:

```python
def calculate_engagement(prev_gaze, current_gaze, emotion, hand_movement, hand_positions, face_positions):
    engagement = 0.5  # Starting baseline
    
    # Customize these weights based on your analysis needs
    if prev_gaze != current_gaze:
        engagement += 0.1  # Gaze movement weight
    
    if emotion == "happy":
        engagement += 0.2  # Positive emotion weight
    elif emotion in ["sad", "angry", "fear"]:
        engagement -= 0.1  # Negative emotion weight
    
    # Add your custom engagement factors here
    
    return max(0, min(engagement, 1))
```

## Output

### Processed Video
- **Visual overlays**: Face bounding boxes, gaze arrows, emotion labels
- **Real-time metrics**: Engagement scores displayed on video
- **Color coding**: Green for child, red for therapist

### Console Output
```
Processing video: 100%|██████████| 1500/1500 [05:23<00:00, 4.64it/s]
Processed video saved to: output_video.mp4
Average Child Engagement: 0.73
Average Therapist Engagement: 0.68
```

## Troubleshooting

### Common Issues

**1. "shape_predictor_68_face_landmarks.dat not found"**
- Download the model file from dlib.net
- Ensure it's in the same directory as your script

**2. Low face detection accuracy**
- Increase `number_of_times_to_upsample` parameter
- Ensure good lighting in your videos
- Try different face detection models (`hog` vs `cnn`)

**3. No hands detected**
- Lower `min_detection_confidence` and `min_tracking_confidence`
- Ensure hands are visible in the video frame

**4. Inaccurate child/therapist identification**
- The script identifies the larger face as the therapist
- Modify `identify_child_and_therapist()` function for your specific use case

### Performance Optimization

**For faster processing**:
```python
# Use HOG model instead of CNN for face detection
face_locations = face_recognition.face_locations(rgb_frame, model="hog")

# Reduce video resolution
# Process every nth frame instead of all frames
```

**For better accuracy**:
```python
# Use CNN model for face detection
face_locations = face_recognition.face_locations(rgb_frame, model="cnn")

# Increase upsampling
number_of_times_to_upsample=3
```

## Advanced Customization

### Adding New Metrics

To add custom engagement metrics, modify the `process_frame()` function:

```python
def process_frame(frame, prev_gaze_child, prev_gaze_therapist):
    # ... existing code ...
    
    # Add your custom analysis here
    custom_metric = calculate_custom_metric(frame_data)
    
    # Display custom metric
    cv2.putText(frame, f"Custom Metric: {custom_metric:.2f}", 
                (10, 110), cv2.FONT_HERSHEY_SIMPLEX, 0.9, (255, 255, 0), 2)
    
    return frame, child_gaze, therapist_gaze, child_engagement, therapist_engagement
```

### Batch Processing Multiple Videos

```python
import glob

video_files = glob.glob("path/to/videos/*.mp4")
for video_file in video_files:
    output_file = video_file.replace(".mp4", "_processed.mp4")
    process_video(video_file, output_file)
```

## File Structure

```
therapy-analysis/
├── therapy_analysis.py
├── requirements.txt
├── shape_predictor_68_face_landmarks.dat
├── input_videos/
│   └── your_video.mp4
└── output_videos/
    └── processed_video.mp4
```

## License

This tool is for research and educational purposes. Ensure you have proper consent and ethical approval before analyzing therapy session videos.

## Contributing

Feel free to modify and improve this tool for your specific research needs. Key areas for enhancement:
- Improved child/therapist identification algorithms
- Additional engagement metrics
- Better emotion recognition models
- Real-time processing capabilities

## Support

For technical issues or questions about adapting this tool for your research, consider:
1. Checking the troubleshooting section above
2. Reviewing the parameter customization options
3. Testing with sample videos first before processing your entire dataset

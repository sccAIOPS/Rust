# Signal Analysis Algorithms

## Overview

Signal analysis algorithms are computational methods designed to extract meaningful information from time-series signals. These algorithms are fundamental to audio processing, communications, biomedical engineering, and many other fields where understanding temporal patterns is crucial.

## Key Concepts

### Signal Types
- **Continuous Signals**: Defined at every point in time (analog)
- **Discrete Signals**: Defined at discrete time intervals (digital)
- **Periodic Signals**: Repeat at regular intervals
- **Aperiodic Signals**: Non-repeating patterns

### Fundamental Properties
- **Frequency**: Number of oscillations per second (Hz)
- **Amplitude**: Signal strength or magnitude
- **Phase**: Position in the cycle at time zero
- **Period**: Time for one complete cycle

### Signal Domains
- **Time Domain**: Signal as function of time
- **Frequency Domain**: Signal decomposed into frequency components (via Fourier Transform)
- **Time-Frequency Domain**: Joint representation (e.g., spectrograms)

## Algorithms in This Category

### YIN Algorithm

**File**: [yin.md](yin.md)

**Purpose**: Fundamental frequency (pitch) estimation from audio signals

**Complexity**: O(N·M) where N is signal length, M is lag range

**Key Innovation**: Cumulative Mean Normalized Difference Function (CMNDF) significantly reduces octave errors compared to traditional autocorrelation methods

**Applications**:
- Music transcription
- Pitch correction (Auto-Tune)
- Instrument tuners
- Speech analysis
- Bioacoustics

**Why It's Important**: 
- One of the most accurate monophonic pitch detection algorithms
- Robust to harmonics and noise
- Used as the foundation for many commercial and research audio tools

## Common Signal Analysis Tasks

### 1. Pitch Detection

**Goal**: Identify the fundamental frequency of a periodic signal

**Algorithms**:
- **YIN** (implemented) - Time-domain, highly accurate
- **Autocorrelation** - Classical approach, prone to octave errors
- **Cepstrum Analysis** - Frequency-domain approach
- **Harmonic Product Spectrum (HPS)** - Multiplication of downsampled spectra
- **pYIN** - Probabilistic extension of YIN

**Applications**: Music transcription, tuning, voice analysis

### 2. Spectral Analysis

**Goal**: Decompose signal into frequency components

**Methods**:
- **Fourier Transform (FFT)** - Convert time → frequency domain
- **Short-Time Fourier Transform (STFT)** - Time-frequency analysis
- **Wavelet Transform** - Multi-resolution time-frequency analysis
- **Mel-Frequency Cepstral Coefficients (MFCC)** - Feature extraction for speech

**Applications**: Audio compression, speech recognition, spectral visualization

### 3. Feature Extraction

**Goal**: Extract meaningful characteristics from signals

**Features**:
- **Temporal**: Zero-crossing rate, energy, duration
- **Spectral**: Centroid, rolloff, flux, bandwidth
- **Cepstral**: MFCC, delta-MFCC
- **Pitch-based**: F0 contour, vibrato rate, jitter, shimmer

**Applications**: Audio classification, speech recognition, instrument identification

### 4. Filtering

**Goal**: Remove unwanted components or enhance desired ones

**Types**:
- **Low-pass**: Remove high frequencies
- **High-pass**: Remove low frequencies  
- **Band-pass**: Keep specific frequency range
- **Band-stop**: Remove specific frequency range
- **Adaptive**: Adjust based on signal properties

**Applications**: Noise reduction, signal enhancement, equalization

### 5. Time-Frequency Analysis

**Goal**: Analyze how frequency content changes over time

**Methods**:
- **Spectrogram**: STFT magnitude visualization
- **Mel Spectrogram**: Perceptually-scaled spectrogram
- **Chromagram**: Pitch class representation
- **Constant-Q Transform**: Logarithmic frequency resolution

**Applications**: Music analysis, speech processing, acoustic event detection

## Audio Signal Processing Pipeline

```
Raw Audio Input
       ↓
1. Preprocessing
   - Normalization
   - DC removal
   - Windowing (Hamming, Hann)
       ↓
2. Feature Extraction
   - Pitch (YIN)
   - Spectral features (FFT)
   - Temporal features
       ↓
3. Analysis/Processing
   - Classification
   - Transcription
   - Synthesis
       ↓
4. Output
   - Processed audio
   - Metadata (notes, chords, etc.)
   - Visualizations
```

## Sampling and Nyquist Theorem

### Nyquist-Shannon Sampling Theorem

**Statement**: To accurately represent a signal containing frequencies up to $f_{max}$, the sampling rate must be at least $2f_{max}$.

**Implications**:
- CD audio: 44.1 kHz sample rate → can represent up to ~22 kHz (human hearing limit)
- Telephone: 8 kHz sample rate → voice up to ~4 kHz
- Ultrasound: 500 kHz+ sample rate → ultrasonic frequencies

**Aliasing**: When sampling rate is too low, high frequencies appear as false low frequencies

### Common Sample Rates

| Sample Rate | Use Case |
|-------------|----------|
| 8 kHz | Telephone, low-quality speech |
| 16 kHz | Wideband speech |
| 22.05 kHz | Low-quality audio |
| 44.1 kHz | CD audio, standard music |
| 48 kHz | Professional audio, video |
| 96 kHz | High-resolution audio |
| 192 kHz | Studio mastering |

## Windowing Functions

When analyzing finite segments of signals, windowing reduces spectral leakage:

### Common Windows

```rust
// Rectangular (no windowing)
fn rectangular(n: usize, N: usize) -> f64 { 1.0 }

// Hamming window
fn hamming(n: usize, N: usize) -> f64 {
    0.54 - 0.46 * (2.0 * PI * n as f64 / (N - 1) as f64).cos()
}

// Hann window
fn hann(n: usize, N: usize) -> f64 {
    0.5 * (1.0 - (2.0 * PI * n as f64 / (N - 1) as f64).cos())
}

// Blackman window
fn blackman(n: usize, N: usize) -> f64 {
    let a0 = 0.42;
    let a1 = 0.5;
    let a2 = 0.08;
    a0 - a1 * (2.0 * PI * n as f64 / (N - 1) as f64).cos()
       + a2 * (4.0 * PI * n as f64 / (N - 1) as f64).cos()
}
```

### Window Properties

| Window | Main Lobe Width | Side Lobe Level | Use Case |
|--------|----------------|-----------------|----------|
| Rectangular | Narrow | High (-13 dB) | Maximum resolution |
| Hamming | Medium | Medium (-43 dB) | General purpose |
| Hann | Medium | Medium (-32 dB) | Smooth, general use |
| Blackman | Wide | Low (-58 dB) | Low leakage needed |

## Real-World Applications

### 1. Music Information Retrieval (MIR)

**Tasks**:
- Beat tracking and tempo estimation
- Chord recognition
- Melody extraction (using YIN)
- Genre classification
- Cover song detection
- Music recommendation

**Datasets**: Million Song Dataset, GTZAN, RWC Music Database

### 2. Speech Processing

**Tasks**:
- Speech recognition (ASR)
- Speaker identification
- Emotion recognition from voice
- Voice activity detection (VAD)
- Pitch tracking for prosody analysis

**Features**: MFCC, pitch (F0), formants, energy

### 3. Audio Effects

**Effects**:
- Pitch shifting (using pitch detection like YIN)
- Time stretching
- Reverb and echo
- Equalization
- Compression and limiting
- Auto-Tune (pitch correction)

### 4. Bioacoustics

**Applications**:
- Bird song analysis
- Whale call detection
- Bat echolocation
- Insect sound analysis

**Challenges**: Wide frequency ranges, environmental noise, overlapping sources

### 5. Medical Signal Analysis

**Signals**:
- Electrocardiogram (ECG)
- Electroencephalogram (EEG)
- Heart sound (phonocardiogram)
- Respiratory signals

**Analysis**: Peak detection, rhythm analysis, anomaly detection

### 6. Telecommunications

**Applications**:
- Voice codecs (speech compression)
- Echo cancellation
- Noise reduction
- Channel equalization

## Performance Considerations

### Computational Complexity

| Operation | Complexity | Notes |
|-----------|-----------|-------|
| FFT | O(N log N) | Fast Fourier Transform |
| DFT | O(N²) | Direct computation |
| Convolution (time) | O(N·M) | Direct method |
| Convolution (freq) | O(N log N) | Using FFT |
| Autocorrelation | O(N²) or O(N log N) | Direct or FFT-based |
| YIN | O(N·M) | M = lag range |

### Real-Time Processing

**Constraints**:
- Processing time < frame duration
- Low latency (< 10 ms for interactive applications)
- Memory limitations on embedded devices

**Optimization Strategies**:
- Use FFT for frequency-domain operations
- Downsample when appropriate
- Buffer management (ring buffers)
- SIMD vectorization
- Parallel processing

### Memory Usage

```
Signal Buffer: N samples × bytes_per_sample
FFT: 2N complex values (4N floats for complex)
Spectrogram: (N/hop_size) × (fft_size/2 + 1) values
```

**Example** (1 second of audio):
- Sample rate: 44100 Hz
- Bit depth: 16-bit (2 bytes)
- Memory: 44100 × 2 = 88.2 KB raw audio

## Signal Quality Metrics

### Signal-to-Noise Ratio (SNR)

$$\text{SNR} = 10 \log_{10} \left( \frac{P_{\text{signal}}}{P_{\text{noise}}} \right) \text{ dB}$$

**Interpretation**:
- SNR > 20 dB: Good quality
- SNR 10-20 dB: Acceptable
- SNR < 10 dB: Poor quality

### Total Harmonic Distortion (THD)

Measures distortion by comparing fundamental to harmonics:

$$\text{THD} = \frac{\sqrt{P_2^2 + P_3^2 + P_4^2 + \ldots}}{P_1}$$

where $P_i$ is the power of the $i$-th harmonic.

## Tools and Libraries

### Rust Ecosystem

- **rustfft**: Fast Fourier Transform
- **dasp**: Digital audio signal processing
- **rodio**: Audio playback
- **cpal**: Cross-platform audio I/O
- **hound**: WAV file I/O

### Python Ecosystem

- **librosa**: Music and audio analysis
- **scipy.signal**: Signal processing
- **numpy/scipy.fft**: FFT implementations
- **soundfile**: Audio I/O
- **pydub**: Audio manipulation

### Other Languages

- **MATLAB/Octave**: Industry standard for signal processing
- **Essentia** (C++): Comprehensive audio analysis
- **Aubio** (C): Real-time audio labeling
- **TarsosDSP** (Java): Audio processing

## Further Reading

### Textbooks

1. **Digital Signal Processing** by Oppenheim & Schafer
   - Classical reference for DSP fundamentals

2. **The Scientist and Engineer's Guide to Digital Signal Processing** by Steven W. Smith
   - Free online: www.dspguide.com

3. **Theory and Applications of Digital Speech Processing** by Rabiner & Schafer
   - Comprehensive speech processing text

4. **Computer Music Tutorial** by Curtis Roads
   - Audio signal processing for music

### Research Papers

1. de Cheveigné, A., & Kawahara, H. (2002). "YIN, a fundamental frequency estimator for speech and music"
   - Original YIN algorithm paper

2. Müller, M. (2015). "Fundamentals of Music Processing"
   - Modern MIR textbook

### Online Resources

- [DSP Related](https://www.dsprelated.com/) - Articles and tutorials
- [Seeing Circles, Sines, and Signals](https://jackschaedler.github.io/circles-sines-signals/) - Interactive visualization
- [Julius O. Smith's Online Books](https://ccrma.stanford.edu/~jos/) - Free DSP textbooks

## Related Topics

- [Math Algorithms](../math/README.md) - FFT, interpolation, numerical methods
- [String Algorithms](../string/README.md) - Pattern matching (related to signal matching)
- [Machine Learning](../machine_learning/README.md) - Audio classification, feature learning

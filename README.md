# Notes in-between

Robust Motion In-betweening
- Most citations by far
- Fills in huge gaps between key-frames
- time-to-arrival embedding
- scheduled target noise
- Recurrent Neural Networks (RNN)
- Long short-term memory (LSTM)

Conditional motion in-betweening
- not always better qualitively than robust
- faster inference
- pose or semantic conditions
- Transformer encoder-based architecture
- Gaussian random path for data augmentation

Single-Shot Motion Completion with Transformer
- Key-frame interpolation only
- Competitive quality
- Real-time inference
- BERT based transformer

Motion In-betweening via Deep ∆-Interpolator
- public code and data
- SLERP Baseline
- local space unlike common global space
- feed the delta of keyframes into the model
- Transformer
  - Separate transformer blocks for key-frames and missing, interpolated frames. Keyframe encoded feeding into missing frame encoder
  - Decoder is MLP

Motion In-betweening via Two-stage Transformers
- Context Transformer give rough poses
- Detail Transformer polish them
- Losses:
  - Local Rotation Loss
  - Global Position Loss
  - Smoothness Loss (time derivative of position)
  - Contact Loss (Foot)
  - Foot Sliding Loss

Generative Tweening: Long-term Inbetweening of 3D Human Motions
- GAN
  - Generator is U-Net without the skip connections (auto-encoder)
  - Discriminator is 7 convolution layers
- predict local motion (1D CNN)
  -> global motion (another 1D CNN)
  - works in local for GAN stability
- inference faster than real-time
- euler angles instead of quaternions
- motion dna for style

A Unified Framework for Real Time Motion Completion
- Transformer + BERT
- positional embedding also trained
- different embeddings for key-frames and predicted frames
- supports input in both local and global space
- input pose vectors to tokens via 1D convolution
- Loss functions:
  - Kinematic:
    - Forward
    - Inverse
  - Motion perceptual
    - reduce the foot-skate
    - discrete wavelet transformation (DWT) for better high-frequency information
- inference real time

Real-time Controllable Motion Transition for Characters
- Conditonal VAE
  - with mixture of experts
- 2 main components:
  - Learns a a natural motion manifold model
  - Sampler is Recurrent Neural Network, predicting next frame
- beats RTN
- inference real time
- extended to support styles with:
  - Real-time Stylized Motion Transition for Characters

Motion In-Betweening with Phase Manifolds
- SOTA imo, especially in edge cases:
  - running too fast -> jump
  - walking backwards
  - quadruped, not only humanoid
- focus on large time gaps
- mixture of experts
  - motion prediction network
  - the gating network
- predict phase updates in frequency form, using periodic auto-encoder
- auto-regressive
- trained in both directions in time (bi-directional)
- contact information in model input
- inference real time
- public source code

Transformer Based Motion In-Betweening
- Not competetive

Let's try:
- Problem: not as smooth at the point of key frames
  - Solution: increase the order of differentiation:
    - Instead of only feeding first order positions and orientations
    - Feed higher order physical parameters:
      - Feed the angular and linear momentums of each limb
    - Loss function to penalize strong forces
    - momentum (not velocity) differentiates fat from thin
    - Before 2005 was utilized, then shifted towards first order
- Diffusion never used, worth a try
  - great for variety by nature without the need of additonal noise



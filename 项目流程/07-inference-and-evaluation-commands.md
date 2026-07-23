

```bash

conda activate so101-real-py312
cd /home/quixoteh/SO-ARM-101

export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
export HF_HUB_CACHE=/home/quixoteh/SO-ARM-101/artifacts/so101-smolvla-training-bundle-2026-07-20/models/hf_cache
export TOKENIZERS_PARALLELISM=false

lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.disable_torque_on_disconnect=true \
  --robot.cameras='{
camera1: {
type: opencv,
index_or_path: "/dev/videoSO101_GRIPPER",
width: 480,
height: 640,
fps: 30,
fourcc: MJPG,
rotation: 90
},
camera2: {
type: opencv,
index_or_path: "/dev/videoSO101_ENV",
width: 640,
height: 480,
fps: 20,
fourcc: YUYV
}
}' \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --dataset.repo_id=local/eval_so101_green_ckpt_010000_recal_v1 \
  --dataset.root=/home/quixoteh/SO-ARM-101/results/so101-smolvla-experiment-2026-07-20/eval_so101_green_ckpt_002000_recal_v1 \
  --dataset.single_task='put small green block in plate' \
  --dataset.fps=20 \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=20 \
  --dataset.num_episodes=10 \
  --dataset.video=true \
  --dataset.push_to_hub=false \
  --dataset.num_image_writer_processes=0 \
  --policy.path=/home/quixoteh/coding/so101-smolvla-experiment-2026-07-20/03_baseline_25000/checkpoints/010000/pretrained_model \
  --policy.device=cuda \
  --policy.load_vlm_weights=false \
  --play_sounds=false
```


```bash
conda activate so101-real-py312
cd /home/quixoteh/SO-ARM-101

export HF_HUB_OFFLINE=1
export TRANSFORMERS_OFFLINE=1
export HF_HUB_CACHE=/home/quixoteh/SO-ARM-101/artifacts/so101-smolvla-training-bundle-2026-07-20/models/hf_cache
export TOKENIZERS_PARALLELISM=false

lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttySO101_FOLLOWER \
  --robot.id=my_follower \
  --robot.disable_torque_on_disconnect=true \
  --robot.cameras='{
camera1: {
type: opencv,
index_or_path: "/dev/videoSO101_GRIPPER",
width: 480,
height: 640,
fps: 30,
fourcc: MJPG,
rotation: 90
},
camera2: {
type: opencv,
index_or_path: "/dev/videoSO101_ENV",
width: 640,
height: 480,
fps: 20,
fourcc: YUYV
}
}' \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttySO101_LEADER \
  --teleop.id=my_leader \
  --dataset.repo_id=local/eval_so101_green_ckpt_010000_recal_v1 \
  --dataset.root=/home/quixoteh/SO-ARM-101/results/so101-smolvla-experiment-2026-07-20/eval_so101_green_ckpt_002000_recal_v1 \
  --dataset.single_task='put small green block in plate' \
  --dataset.fps=20 \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=20 \
  --dataset.num_episodes=13 \
  --dataset.video=true \
  --dataset.push_to_hub=false \
  --dataset.num_image_writer_processes=0 \
  --resume=true \
  --policy.path=/home/quixoteh/coding/so101-smolvla-experiment-2026-07-20/03_baseline_25000/checkpoints/010000/pretrained_model \
  --policy.device=cuda \
  --policy.load_vlm_weights=false \
  --play_sounds=false
```
# Neuroevolution of Recurrent Architectures

### Installation (tested on Ubuntu 20.04)

```
# Debian packages                       ----------mpi4py---------- -Box2D-- ------MuJoCo-----------
sudo apt install git python3-virtualenv python3-dev libopenmpi-dev g++ swig libosmesa6-dev patchelf

# MuJoCo
wget https://mujoco.org/download/mujoco210-linux-x86_64.tar.gz -P ~/Downloads/
mkdir -p ~/.mujoco/ && tar -zxf ~/Downloads/mujoco210-linux-x86_64.tar.gz -C ~/.mujoco/
echo -e "\n# MuJoCo\nMUJOCO_PATH=/home/max/.mujoco/mujoco210/bin
export LD_LIBRARY_PATH=\$MUJOCO_PATH:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc

# Paper Code & Dependencies
git clone https://github.com/neuroevolution-recurrent-architectures/repo && cd repo/
virtualenv ~/nra_venv && source ~/nra_venv/bin/activate && pip3 install -r requirements.txt
```

### Execution

`<task>` = `acrobot`, `cart_pole`, `mountain_car`, `mountain_car_continuous`, `pendulum`, `bipedal_walker`, `bipedal_walker_hardcore`, `lunar_lander`, `lunar_lander_continuous`, `ant`, `half_cheetah`, `hopper`, `humanoid`, `humanoid_standup`, `inverted_double_pendulum`, `inverted_pendulum`, `reacher`, `swimmer` or `walker_2d` | `<net>` = `static/rnn` or `dynamic/rnn` | `<N>` for the number of MPI processes | `<G>` for the number of generations | `<S>` for the population size

```
mpiexec -n <N> python3 main.py --env_path envs/control/score.py \
                               --bots_path bots/<net>/control.py \
                               --nb_generations <G> \
                               --population_size <S> \
                               --additional_arguments '{"task" : "<task>"}'
```

Example from the paper: (you might need to decrease the number of MPI processes on older computers)
```
mpiexec -n 16 python3 main.py --env_path envs/control/score.py \
                              --bots_path bots/dynamic/rnn/control.py \
                              --nb_generations 100 \
                              --population_size 16 \
                              --additional_arguments '{"task" : "acrobot"}'
```

### Downloading the Paper's Results & Final Dynamic States

```
wget https://www.dropbox.com/s/srigcl1dds5n833/envs.control.score.zip -P ~/Downloads/
unzip -o ~/Downloads/envs.control.score.zip -d data/states/
```

You can now run additional generations ...
```
mpiexec -n 16 python3 main.py --env_path envs/control/score.py \
                              --bots_path bots/dynamic/rnn/control.py \
                              --nb_elapsed_generations 100 \
                              --nb_generations 100 \
                              --population_size 16 \
                              --additional_arguments '{"task" : "cart_pole"}'
```

... Evaluate the new agents ...
```
mpiexec -n 1 python3 utils/evaluate.py --state_path data/states/envs.control.score/task.cart_pole/bots.dynamic.rnn.control/16/
```

... And both record the elite's behaviour and obtain its architecture.
```
python3 utils/record.py --state_path data/states/envs.control.score/task.cart_pole/bots.dynamic.rnn.control/16/200/
```

### Reproducing the Paper's Figures

```
# Verify Stable Baselines 3 Results (not necessary for later steps)
wget https://www.dropbox.com/s/448w1lznuljbh5b/models.zip -P ~/Downloads/
unzip ~/Downloads/models.zip -d data/
jupyter notebook utils/notebooks/sb3.ipynb

# Visualize the dynamic networks
jupyter notebook utils/notebooks/drnn.ipynb

# Reproduce the paper's figures  
jupyter notebook utils/notebooks/control.ipynb
```
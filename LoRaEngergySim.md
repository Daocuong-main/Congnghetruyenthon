```diff
[1mdiff --git a/Framework/Node.py b/Framework/Node.py[m
[1mindex 1c82dd3..25e36fa 100644[m
[1m--- a/Framework/Node.py[m
[1m+++ b/Framework/Node.py[m
[36m@@ -33,7 +33,7 @@[m [mclass Node:[m
                  massive_mimo_gain=False, number_of_antennas=1):[m
         self.power_gain = 1[m
         if massive_mimo_gain:[m
[31m-            self.power_gain = 1/np.sqrt(number_of_antennas)[m
[32m+[m[32m            self.power_gain = 1 / np.sqrt(number_of_antennas)[m
         self.num_tx_state_changes = 0[m
         self.total_wait_time_because_dc = 0[m
         self.num_no_downlink = 0[m
[36m@@ -47,43 +47,37 @@[m [mclass Node:[m
         self.energy_profile = energy_profile[m
         self.base_station = base_station[m
         self.process_time = process_time[m
[31m-        # self.air_interface = AirInterface(base_station)[m
         self.env = env[m
         self.stop_state_time = self.env.now[m
         self.start_state_time = self.env.now[m
         self.current_state = NodeState.OFFLINE[m
         self.lora_param = lora_parameters[m
         self.payload_size = payload_size[m
[31m-[m
         self.prev_power_mW = 0[m
[31m-[m
         self.air_interface = air_interface[m
[31m-[m
         self.location = location[m
[31m-[m
         self.sleep_time = sleep_time[m
[31m-[m
         self.change_lora_param = dict()[m
         self.energy_value = 0[m
[31m-[m
         self.lost_packages_time = [][m
[31m-[m
         self.power_tracking = {'val': [], 'time': []}[m
         self.energy_measurements = {'val': [], 'time': []}[m
         self.state_changes = {'val': [], 'time': []}[m
[31m-        self.energy_tracking = {NodeState(NodeState.SLEEP).name: 0.0, NodeState(NodeState.PROCESS).name: 0.0,[m
[31m-                                NodeState(NodeState.RX).name: 0.0, NodeState(NodeState.TX).name: 0.0}[m
[31m-[m
[32m+[m[32m        # Modified: Initialize energy_tracking with additional keys for join states.[m
[32m+[m[32m        self.energy_tracking = {[m
[32m+[m[32m            NodeState.SLEEP.name: 0.0,[m
[32m+[m[32m            NodeState.PROCESS.name: 0.0,[m
[32m+[m[32m            NodeState.RX.name: 0.0,[m
[32m+[m[32m            NodeState.TX.name: 0.0,[m
[32m+[m[32m            NodeState.JOIN_TX.name: 0.0,[m
[32m+[m[32m            NodeState.JOIN_RX.name: 0.0[m
[32m+[m[32m        }[m
         self.bytes_sent = 0[m
[31m-[m
         self.packet_to_sent = None[m
[31m-[m
         self.time_off = dict()[m
         for ch in LoRaParameters.CHANNELS:[m
             self.time_off[ch] = 0[m
[31m-[m
         self.confirmed_messages = confirmed_messages[m
[31m-[m
         self.unique_packet_id = 0[m
 [m
     def plot(self, prop_measurements):[m
[36m@@ -123,94 +117,77 @@[m [mclass Node:[m
         plt.show()[m
 [m
     def run(self):[m
[31m-        random_wait = np.random.uniform(0,  MAX_DELAY_START_PER_NODE_MS)[m
[32m+[m[32m        random_wait = np.random.uniform(0, MAX_DELAY_START_PER_NODE_MS)[m
         yield self.env.timeout(random_wait)[m
         self.start_device_active = self.env.now[m
[31m-        if  PRINT_ENABLED:[m
[32m+[m[32m        if PRINT_ENABLED:[m
             print('{} ms delayed prior to joining'.format(random_wait))[m
             print('{} joining the network'.format(self.id))[m
[31m-            # TODO ERROR!!!!! self.process[m
[31m-            self.join(self.env)[m
[31m-        if  PRINT_ENABLED:[m
[32m+[m[32m            yield self.env.process(self.join(self.env))  # FIX: Yield the join process[m
[32m+[m[32m        if PRINT_ENABLED:[m
             print('{}: joined the network'.format(self.id))[m
         while True:[m
[31m-            # added also a random wait to accommodate for any timing issues on the node itself[m
[31m-            random_wait = np.random.randint(0,  MAX_DELAY_BEFORE_SLEEP_MS)[m
[32m+[m[32m            random_wait = np.random.randint(0, MAX_DELAY_BEFORE_SLEEP_MS)[m
             yield self.env.timeout(random_wait)[m
[31m-[m
             yield self.env.process(self.sleep())[m
[31m-[m
             yield self.env.process(self.processing())[m
[31m-            # after processing go back to sleep[m
             self.track_power(self.energy_profile.sleep_power_mW)[m
[31m-[m
[31m-            # ------------SENDING------------ #[m
[31m-            if  PRINT_ENABLED:[m
[32m+[m[32m            if PRINT_ENABLED:[m
                 print('{}: SENDING packet'.format(self.id))[m
[31m-[m
             self.unique_packet_id += 1[m
[31m-[m
             payload_size = self.payload_size[m
             if MAC_IMPROVEMENT and self.packets_sent < 20:[m
                 payload_size = 5[m
[31m-[m
             packet = UplinkMessage(node=self, start_on_air=self.env.now, payload_size=payload_size,[m
                                    confirmed_message=self.confirmed_messages, id=self.unique_packet_id)[m
             downlink_message = yield self.env.process(self.send(packet))[m
             if downlink_message is None:[m
[31m-                # message is collided and not received at the BS[m
                 yield self.env.process(self.dl_message_lost())[m
             else:[m
                 yield self.env.process(self.process_downlink_message(downlink_message, packet))[m
[31m-[m
[31m-            if  PRINT_ENABLED:[m
[32m+[m[32m            if PRINT_ENABLED:[m
                 print('{}: DONE sending'.format(self.id))[m
[31m-[m
[31m-            self.num_unique_packets_sent += 1  # at the end to be sure that this packet was tx[m
[32m+[m[32m            self.num_unique_packets_sent += 1[m
 [m
     # [----JOIN----]        [rx1][m
     # computes time spent in different states during join procedure[m
     # TODO also allow join reqs to be collided[m
     def join(self, env):[m
[31m-[m
[31m-        self.join_tx()[m
[31m-[m
[31m-        self.join_wait()[m
[31m-[m
[31m-        self.join_rx()[m
[32m+[m[32m        yield self.env.process(self.join_tx())[m
[32m+[m[32m        yield self.env.process(self.join_wait())[m
[32m+[m[32m        yield self.env.process(self.join_rx())[m
         return True[m
 [m
     def join_tx(self):[m
[31m-[m
[31m-        if  PRINT_ENABLED:[m
[32m+[m[32m        if PRINT_ENABLED:[m
             print('{}: \t JOIN TX'.format(self.id))[m
         energy = LoRaParameters.JOIN_TX_ENERGY_MJ[m
[31m-[m
         power = (LoRaParameters.JOIN_TX_ENERGY_MJ / LoRaParameters.JOIN_TX_TIME_MS) * 1000[m
         self.track_power(power)[m
         yield self.env.timeout(LoRaParameters.JOIN_TX_TIME_MS)[m
         self.track_power(power)[m
[31m-        self.track_energy('tx', energy)[m
[32m+[m[32m        # Use NodeState.JOIN_TX instead of string 'tx'[m
[32m+[m[32m        self.track_energy(NodeState.JOIN_TX, energy)[m
 [m
     def join_wait(self):[m
[31m-        if  PRINT_ENABLED:[m
[32m+[m[32m        if PRINT_ENABLED:[m
             print('{}: \t JOIN WAIT'.format(self.id))[m
         self.track_power(self.energy_profile.sleep_power_mW)[m
         yield self.env.timeout(LoRaParameters.JOIN_ACCEPT_DELAY1)[m
         energy = LoRaParameters.JOIN_ACCEPT_DELAY1 * self.energy_profile.sleep_power_mW[m
[31m-[m
         self.track_power(self.energy_profile.sleep_power_mW)[m
[31m-        self.track_energy('sleep', energy)[m
[32m+[m[32m        # Use NodeState.SLEEP for waiting[m
[32m+[m[32m        self.track_energy(NodeState.SLEEP, energy)[m
 [m
     def join_rx(self):[m
[31m-        # TODO RX1 and RX2[m
[31m-        if  PRINT_ENABLED:[m
[32m+[m[32m        if PRINT_ENABLED:[m
             print('{}: \t JOIN RX'.format(self.id))[m
         power = (LoRaParameters.JOIN_RX_ENERGY_MJ / LoRaParameters.JOIN_RX_TIME_MS) * 1000[m
         self.track_power(power)[m
         yield self.env.timeout(LoRaParameters.JOIN_RX_TIME_MS)[m
         self.track_power(power)[m
[31m-        self.track_energy('rx', LoRaParameters.JOIN_RX_ENERGY_MJ)[m
[32m+[m[32m        # Use NodeState.JOIN_RX instead of string 'rx'[m
[32m+[m[32m        self.track_energy(NodeState.JOIN_RX, LoRaParameters.JOIN_RX_ENERGY_MJ)[m
 [m
     # [----transmit----]        [rx1]      [--rx2--][m
     # computes time spent in different states during tx and rx one package[m
[36m@@ -501,12 +478,18 @@[m [mclass Node:[m
             self.current_state = new_state[m
 [m
     def energy_per_bit(self) -> float:[m
[32m+[m[32m        if self.packets_sent == 0 or self.payload_size == 0:[m
[32m+[m[32m            return 0.0[m
         return self.total_energy_consumed() / (self.packets_sent * self.payload_size * 8)[m
 [m
     def transmit_related_energy_per_bit(self) -> float:[m
[32m+[m[32m        if self.packets_sent == 0 or self.payload_size == 0:[m
[32m+[m[32m            return 0.0[m
         return self.transmit_related_energy_consumed() / (self.packets_sent * self.payload_size * 8)[m
 [m
     def transmit_related_energy_per_unique_bit(self) -> float:[m
[32m+[m[32m        if self.num_unique_packets_sent == 0 or self.payload_size == 0:[m
[32m+[m[32m            return 0.0[m
         return self.transmit_related_energy_consumed() / (self.num_unique_packets_sent * self.payload_size * 8)[m
 [m
     def transmit_related_energy_consumed(self) -> float:[m
[1mdiff --git a/Simulations/Example/simulation.py b/Simulations/Example/simulation.py[m
[1mindex 4ff8cb1..8879499 100644[m
[1m--- a/Simulations/Example/simulation.py[m
[1m+++ b/Simulations/Example/simulation.py[m
[36m@@ -3,10 +3,10 @@[m [mimport math[m
 import multiprocessing as mp[m
 import os[m
 import pickle[m
[31m-[m
 import pandas as pd[m
 [m
[31m-import SimulationProcess[m
[32m+[m[32mfrom . import SimulationProcess[m
[32m+[m
 from Simulations.GlobalConfig import *[m
 from Framework import Location as loc[m
 [m
[36m@@ -44,7 +44,8 @@[m [mif __name__ == '__main__':[m
     """[m
     In this simulation, we want to see the effect of payload size on different paramaters[m
     """[m
[31m-[m
[32m+[m[32m    current_dir = os.path.dirname(os.path.abspath(__file__))[m
[32m+[m[32m    locations_file = os.path.join(current_dir, "Locations", "50_locations_1000_sim.pkl")[m
     # Load generated locations[m
     with open(locations_file, 'rb') as file_handler:[m
         locations_per_simulation = pickle.load(file_handler)[m
[36m@@ -86,7 +87,7 @@[m [mif __name__ == '__main__':[m
     # create a Processing pool to speed-up the process[m
     # we are not greedy and only use 20% of the available CPUs[m
     # change this to your liking[m
[31m-    pool = mp.Pool(math.floor(mp.cpu_count() / 5))[m
[32m+[m[32m    pool = mp.Pool(mp.cpu_count())[m
     for n_sim in range(num_of_simulations):[m
         print(f'Simulation #{n_sim}')[m
         locations = locations_per_simulation[n_sim][m
[1mdiff --git a/Simulations/GlobalConfig.py b/Simulations/GlobalConfig.py[m
[1mindex 6dd343c..d80a322 100644[m
[1m--- a/Simulations/GlobalConfig.py[m
[1m+++ b/Simulations/GlobalConfig.py[m
[36m@@ -1,3 +1,4 @@[m
[32m+[m[32m# ----- Updated File: Simulations/GlobalConfig.py -----[m
 import numpy as np[m
 [m
 ############### SIMULATION SPECIFIC PARAMETERS ###############[m
[36m@@ -6,8 +7,9 @@[m [mstart_sf = 7[m
 [m
 scaling_factor = 0.1[m
 transmission_rate_id = str(scaling_factor)[m
[31m-transmission_rate_bit_per_ms = scaling_factor*(12*8)/(60*60*1000)  # 12*8 bits per hour (1 typical packet per hour)[m
[31m-simulation_time = 24 * 60 * 60 * 1000 * 30/scaling_factor[m
[32m+[m[32mtransmission_rate_bit_per_ms = scaling_factor * (12 * 8) / (60 * 60 * 1000)  # 12*8 bits per hour[m
[32m+[m[32m# Set simulation_time to 4 hours (in ms) for the experiment[m
[32m+[m[32msimulation_time = 4 * 60 * 60 * 1000[m[41m  [m
 cell_size = 1000[m
 adr = True[m
 confirmed_messages = True[m
[36m@@ -17,11 +19,12 @@[m [mpath_loss_variances = [7.9]  # [0, 5, 7.8, 15, 20][m
 [m
 MAC_IMPROVEMENT = False[m
 num_locations = 50[m
[31m-num_of_simulations = 1000[m
[31m-locations_file = "locations/"+"{}_locations_{}_sim.pkl".format(num_locations, num_of_simulations)[m
[31m-results_file = "results/{}_{}_{}_cnst_num_bytes.p".format(adr, confirmed_messages, transmission_rate_id)[m
[32m+[m[32mnum_of_simulations = 1  # For experiment redeployment, use 1 simulation run[m
[32m+[m[32mlocations_file = "locations/" + "{}_locations_{}_sim.pkl".format(num_locations, num_of_simulations)[m
[32m+[m[32mresults_file = "results/extended_{}_{}_{}_cnst_num_bytes.p".format(adr, confirmed_messages, transmission_rate_id)[m
 [m
[31m-############### SIMULATION SPECIFIC PARAMETERS ###############[m
[32m+[m[32m# NEW: Extended simulation parameter for multi-gateway experiment[m
[32m+[m[32mmulti_gateway = True[m
 [m
 ############### DEFAULT PARAMETERS ###############[m
 LOG_ENABLED = True[m
[36m@@ -30,6 +33,4 @@[m [mPRINT_ENABLED = True[m
 MAX_DELAY_START_PER_NODE_MS = np.round(simulation_time / 10)[m
 track_changes = True[m
 middle = np.round(cell_size / 2)[m
[31m-load_prev_simulation_results = True[m
[31m-[m
[31m-############### DEFAULT PARAMETERS ###############[m
[32m+[m[32mload_prev_simulation_results = False[m
[1mdiff --git a/Simulations/channel_var/generate_locations.py b/Simulations/channel_var/generate_locations.py[m
[1mindex 48e781b..e6fd320 100644[m
[1m--- a/Simulations/channel_var/generate_locations.py[m
[1m+++ b/Simulations/channel_var/generate_locations.py[m
[36m@@ -6,7 +6,7 @@[m [mfrom Location import Location[m
 [m
 num_locations = 500[m
 cell_size = 1000[m
[31m-num_of_simulations = 1000[m
[32m+[m[32mnum_of_simulations = 1[m
 [m
 locations_per_simulation = list()[m
 [m
[1mdiff --git a/Simulations/channel_var/simulation.py b/Simulations/channel_var/simulation.py[m
[1mindex adf9368..9131540 100644[m
[1m--- a/Simulations/channel_var/simulation.py[m
[1m+++ b/Simulations/channel_var/simulation.py[m
[36m@@ -80,7 +80,7 @@[m [mif __name__ == '__main__':[m
         _results['mean_energy'][str(payload_size)] = dict()[m
         _results['std_energy'][str(payload_size)] = dict()[m
 [m
[31m-    pool = mp.Pool(math.floor(mp.cpu_count() / 5))[m
[32m+[m[32m    pool = mp.Pool(mp.cpu_count())[m
     for n_sim in range(resume_from_simulation, num_of_simulations):[m
         print('Simulation #{}'.format(n_sim))[m
         locations = locations_per_simulation[n_sim][m
[1mdiff --git a/Simulations/load_variances/simulation.py b/Simulations/load_variances/simulation.py[m
[1mindex adf9368..9131540 100644[m
[1m--- a/Simulations/load_variances/simulation.py[m
[1m+++ b/Simulations/load_variances/simulation.py[m
[36m@@ -80,7 +80,7 @@[m [mif __name__ == '__main__':[m
         _results['mean_energy'][str(payload_size)] = dict()[m
         _results['std_energy'][str(payload_size)] = dict()[m
 [m
[31m-    pool = mp.Pool(math.floor(mp.cpu_count() / 5))[m
[32m+[m[32m    pool = mp.Pool(mp.cpu_count())[m
     for n_sim in range(resume_from_simulation, num_of_simulations):[m
         print('Simulation #{}'.format(n_sim))[m
         locations = locations_per_simulation[n_sim][m

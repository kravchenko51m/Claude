# GEPRC Cinelog35 V2 — Betaflight Config Reference

Personal reference for the ongoing PID/rate tuning of a GEPRC Cinelog35 V2. This file is updated whenever the live FC config changes.

## Hardware / Firmware

- Frame: GEPRC Cinelog35 V2
- FC: GEPRC F722 AIO (ICM42688P gyro), `board_name = GEPRC_F722_AIO`
- Battery: 6S primary, occasionally 4S
- VTX/RX: DJI O3 air unit (digital FPV, no analog goggles)
- GPS: present — used only for crash-location display on OSD (GPS Rescue not enabled, no plans to enable)
- Camera: no GoPro
- Firmware: Betaflight 4.5.0-RC3 (Mar 18 2024 build), config rev `415237c`
- Connects via COM4 (USB)

## Current Configuration Summary (as of 2026-06-15)

### PID profiles (auto-selected by battery cell count)

| Profile | Name | `auto_profile_cell_count` | Notes |
|---|---|---|---|
| 0 | 6S | 6 | Original GEPRC "simplified tuning" PIDs + `thrust_linear=12` |
| 1 | 4S | 4 | Same shape as profile 0, PIDs scaled ~+10%, `thrust_linear=12` |

Betaflight auto-selects the PID profile based on detected battery cell count — no manual switch needed. Profile 1 was previously an orphaned, unreachable "6S+GoPro" profile (no GoPro is used on this craft).

### Rate profiles (selected by AUX4 / channel 8)

| Rate profile | Name | AUX4 position | Settings |
|---|---|---|---|
| 0 | Acro | low | `thr_mid/thr_expo=35`, `roll_expo=10`, `pitch_expo=10`, `yaw_expo=15` (stock rates, softened with expo) |
| 1 | (unused) | middle third — unreachable by a 2-pos switch | plain Betaflight defaults |
| 2 | Cine | high | `thr_mid/thr_expo=35`, `rc_rate=6`, `expo=25`, `srate=45` on roll/pitch/yaw (~450-500°/s max, smooth framing) |

Mechanism: `adjrange 0 0 0 900 2100 12 3 0 0` — RATE_PROFILE adjustment (function 12), always enabled (AUX1 full range as the enable condition), AUX4 (`auxSwitchChannelIndex=3`) is the select channel. Betaflight hardcodes a 3-way split of the select channel's 900-2100µs range to rateprofile 0/1/2, so a normal 2-position switch lands on the outer two (0 and 2), skipping the middle slot (1).

### AUX channel map

| Channel | Function |
|---|---|
| AUX1 (ch5) | ARM (1750-2100) |
| AUX2 (ch6) | Flight mode — ANGLE (900-1625) + PREARM (1300-2100), likely 3-position |
| AUX3 (ch7) | Beeper (BOXBEEPERON, 1700-2100) |
| AUX4 (ch8) | Acro/Cine rate profile switch, 2-position (see above) |

### Other notable settings

- `motor_pwm_protocol = DSHOT600` (was DSHOT300); bidirectional DSHOT + RPM filtering already active
- `thrust_linear = 12` on both PID profiles
- `dyn_notch_count = 2`, `pid_process_denom = 2`

## Changelog

### 2026-06-15
- Repurposed orphaned PID profile 1 ("6S+GoPro", unreachable) into a "4S" profile: PIDs scaled ~+10% over profile 0, auto-selected via `auto_profile_cell_count` (4 vs 6) instead of a manual switch.
- Added `thrust_linear=12` to both PID profiles.
- Softened rate profile 0 (Acro) with expo (roll/pitch 10, yaw 15) — previously 0-expo/twitchy.
- Created "Cine" rates in rate profile 2 (~450-500°/s max rate, 25% expo).
- Switched `motor_pwm_protocol` to DSHOT600 (not yet bench-tested).
- Wired Acro/Cine rate switching to AUX4 (channel 8, 2-position switch) via `adjrange 0 0 0 900 2100 12 3 0 0`. (A competing adjustment briefly added via Configurator GUI on AUX3 was reverted — AUX3's high range is already Beeper, and its enable-range was a dead zone.)
- **Fixed PID profile auto-switching (was completely inactive)**: after the session's earlier CLI work, the *active* PID profile had ended up as profile 2 (empty bench-fallback profile, `auto_profile_cell_count=0`). In Betaflight, `auto_profile_cell_count=0` means "STAY — never auto-switch away from this profile", so the cell-count check was silently exiting every time regardless of battery (confirmed stuck on GUI "Profile 3" with 6S, 4S, and USB all tried). Fix: with a 4S pack connected, ran `profile 1` (the "4S" profile, `auto_profile_cell_count=4`) + `save`. Verified via CLI: `profile` now returns `1` and `status` shows "4S battery - OK". Since profile 0 (count=6) and profile 1 (count=4) both now have valid non-zero counts, future 6S/4S connections will always find an exact match and switch correctly — self-sustaining, no further action needed for normal 4S/6S use.
- **Confirmed working both directions** (Configurator GUI is 1-indexed): 6S battery → GUI "Profile 1" (= CLI profile0 = "6S" tune); 4S battery → GUI "Profile 2" (= CLI profile1 = "4S" tune). PID auto-switching fully verified.

## Known gotchas

- **`auto_profile_cell_count=0` = "STAY"**: if the *active* PID profile ever has `auto_profile_cell_count=0` (true for unused profiles 2/3 by default), Betaflight's cell-count auto-switch permanently no-ops, regardless of battery. To check: CLI `profile` (active index) + `get auto_profile_cell_count`. To fix: `profile <0 or 1>` (whichever matches the connected battery) + `save`. This can only resurface if a battery with a cell count other than 4 or 6 is ever connected (search falls back to a STAY profile) — not expected with this craft's normal 4S/6S use.
- The cell-count check only runs on a battery presence transition (disconnected→connected, voltage stable) while disarmed — it won't re-evaluate mid-flight or while already powered.

## Pending / TODO

- DSHOT600 bench test (motors spin, props off — confirm no desync) before first flight
- Test flight + blackbox review (2026-06-16) — feedback on 4S/6S PID profiles and Acro/Cine rate feel via AUX4
- Firmware update to latest stable (2026.06) — deferred. Plan: full chip erase + flash latest + restore config from the snapshot below + recalibrate accelerometer. Consider GPS-based Position/Altitude Hold (new in 2026.06) given the craft has GPS.

## Full config snapshot (`diff all`, post-change, 2026-06-15)

```
# version
# Betaflight / STM32F7X2 (S7X2) 4.5.0 Mar 18 2024 / 14:15:30 (3aabaf365) MSP API: 1.46
# config rev: 415237c

# start the command batch
batch start

# reset configuration to default settings
defaults nosave

board_name GEPRC_F722_AIO
manufacturer_id GEPR
mcu_id 003200235632501820303236
signature 

# name: Cinelog35 V2

# feature
feature -AIRMODE
feature TELEMETRY
feature OSD

# serial
serial 0 131073 115200 57600 0 115200
serial 1 64 115200 57600 0 115200
serial 4 2 115200 115200 0 115200

# beacon
beacon RX_LOST
beacon RX_SET

# map
map TAER1234

# aux
aux 0 0 0 1750 2100 0 0
aux 1 1 1 900 1625 0 0
aux 2 13 2 1700 2100 0 0
aux 3 28 1 1300 2100 0 0

# adjrange
adjrange 0 0 0 900 2100 12 3 0 0

# master
set dyn_notch_count = 2
set acc_calibration = 11,5,-25,1
set dshot_bidir = ON
set motor_output_reordering = 2,3,0,1,4,5,6,7
set failsafe_delay = 6
set failsafe_switch_mode = STAGE2
set failsafe_recovery_delay = 10
set align_board_yaw = 0
set vbat_max_cell_voltage = 420
set beeper_dshot_beacon_tone = 3
set yaw_motors_reversed = ON
set small_angle = 180
set gps_sbas_mode = AUTO
set gps_set_home_point_once = ON
set gps_rescue_min_start_dist = 30
set gps_rescue_descent_dist = 50
set gps_rescue_descend_rate = 120
set gps_rescue_disarm_threshold = 25
set gps_rescue_min_sats = 5
set deadband = 1
set yaw_deadband = 1
set pid_process_denom = 2
set osd_warn_bitmask = 8191
set osd_vbat_pos = 234
set osd_rssi_pos = 234
set osd_link_quality_pos = 2112
set osd_link_tx_power_pos = 234
set osd_rssi_dbm_pos = 234
set osd_rsnr_pos = 234
set osd_tim_1_pos = 234
set osd_tim_2_pos = 2103
set osd_remaining_time_estimate_pos = 234
set osd_flymode_pos = 2125
set osd_anti_gravity_pos = 234
set osd_g_force_pos = 234
set osd_throttle_pos = 2157
set osd_vtx_channel_pos = 234
set osd_crosshairs_pos = 205
set osd_ah_sbar_pos = 206
set osd_ah_pos = 78
set osd_current_pos = 234
set osd_mah_drawn_pos = 234
set osd_wh_drawn_pos = 234
set osd_motor_diag_pos = 234
set osd_craft_name_pos = 2090
set osd_pilot_name_pos = 234
set osd_gps_speed_pos = 2130
set osd_gps_lon_pos = 2048
set osd_gps_lat_pos = 2065
set osd_gps_sats_pos = 2144
set osd_home_dir_pos = 2318
set osd_home_dist_pos = 2168
set osd_flight_dist_pos = 234
set osd_compass_bar_pos = 234
set osd_altitude_pos = 2135
set osd_pid_roll_pos = 234
set osd_pid_pitch_pos = 234
set osd_pid_yaw_pos = 234
set osd_debug_pos = 234
set osd_power_pos = 234
set osd_pidrate_profile_pos = 234
set osd_warnings_pos = 14667
set osd_avg_cell_voltage_pos = 2080
set osd_pit_ang_pos = 234
set osd_rol_ang_pos = 234
set osd_battery_usage_pos = 234
set osd_disarmed_pos = 234
set osd_nheading_pos = 234
set osd_up_down_reference_pos = 205
set osd_ready_mode_pos = 234
set osd_nvario_pos = 164
set osd_esc_tmp_pos = 234
set osd_esc_rpm_pos = 234
set osd_esc_rpm_freq_pos = 234
set osd_rtc_date_time_pos = 234
set osd_adjustment_range_pos = 234
set osd_flip_arrow_pos = 2222
set osd_core_temp_pos = 234
set osd_log_status_pos = 234
set osd_stick_overlay_left_pos = 234
set osd_stick_overlay_right_pos = 234
set osd_rate_profile_name_pos = 234
set osd_pid_profile_name_pos = 234
set osd_profile_name_pos = 234
set osd_rcchannels_pos = 234
set osd_camera_frame_pos = 35
set osd_efficiency_pos = 234
set osd_total_flights_pos = 234
set osd_aux_pos = 234
set osd_sys_goggle_voltage_pos = 234
set osd_sys_vtx_voltage_pos = 234
set osd_sys_bitrate_pos = 234
set osd_sys_delay_pos = 234
set osd_sys_distance_pos = 234
set osd_sys_lq_pos = 234
set osd_sys_goggle_dvr_pos = 234
set osd_sys_vtx_dvr_pos = 234
set osd_sys_warnings_pos = 234
set osd_sys_vtx_temp_pos = 234
set osd_sys_fan_speed_pos = 234
set osd_gps_sats_show_hdop = ON
set osd_canvas_width = 30
set osd_canvas_height = 13
set debug_mode = GYRO_SCALED
set vcd_video_system = AUTO
set displayport_msp_fonts = 0,0,0,0
set gyro_1_sensor_align = CW90FLIP
set gyro_1_align_pitch = 1800
set gyro_2_bustype = SPI
set gyro_2_sensor_align = CW0
set craft_name = Cinelog35 V2

profile 0

# profile 0
set profile_name = 6S
set dterm_lpf1_dyn_min_hz = 71
set dterm_lpf1_dyn_max_hz = 142
set dterm_lpf1_static_hz = 71
set dterm_lpf2_static_hz = 142
set p_pitch = 51
set i_pitch = 110
set d_pitch = 40
set f_pitch = 106
set p_roll = 49
set i_roll = 105
set d_roll = 35
set f_roll = 101
set p_yaw = 49
set i_yaw = 105
set f_yaw = 101
set horizon_level_strength = 50
set d_min_roll = 35
set d_min_pitch = 40
set auto_profile_cell_count = 6
set thrust_linear = 12
set simplified_i_gain = 120
set simplified_d_gain = 120
set simplified_pi_gain = 110
set simplified_dmax_gain = 0
set simplified_feedforward_gain = 85
set simplified_dterm_filter_multiplier = 95
set ez_landing_limit = 5

profile 1

# profile 1
set profile_name = 4S
set dterm_lpf1_dyn_min_hz = 71
set dterm_lpf1_dyn_max_hz = 142
set dterm_lpf1_static_hz = 71
set dterm_lpf2_static_hz = 142
set p_pitch = 56
set i_pitch = 121
set d_pitch = 44
set f_pitch = 117
set p_roll = 54
set i_roll = 116
set d_roll = 39
set f_roll = 111
set p_yaw = 54
set i_yaw = 116
set f_yaw = 111
set horizon_level_strength = 50
set d_min_roll = 39
set d_min_pitch = 44
set auto_profile_cell_count = 4
set thrust_linear = 12
set simplified_pids_mode = OFF
set ez_landing_limit = 5

profile 2

profile 3

# restore original profile selection
profile 2

rateprofile 0

# rateprofile 0
set thr_mid = 35
set thr_expo = 35
set roll_expo = 10
set pitch_expo = 10
set yaw_expo = 15

rateprofile 1

rateprofile 2

# rateprofile 2
set thr_mid = 35
set thr_expo = 35
set roll_rc_rate = 6
set pitch_rc_rate = 6
set yaw_rc_rate = 6
set roll_expo = 25
set pitch_expo = 25
set yaw_expo = 25
set roll_srate = 45
set pitch_srate = 45
set yaw_srate = 45

rateprofile 3

# restore original rateprofile selection
rateprofile 2

# save configuration
save
```

# Logs for relevant info

## 5/12: Looking at features: [station id, exact arrival timestamp, exact departure timestamp, headway, dwell time, train-to-train spacing, per station delay propagation, sequence modeling]

The code looks clean and correct. Here's a summary of what the cell does and the key findings:

What already exists in your data:

station_id — stop_id exists in both static and realtime datasets, ready to use
stop_sequence — exists in stop_times.txt (static) and current_stop_sequence (realtime), forms the basis for sequence modeling
What the code computes from static schedule:

Scheduled arrival/departure timestamps — combines stop_times.arrival_time (HH:MM:SS) with the date to produce full datetimes
Scheduled headway — diffs consecutive arrivals at the same stop/route/direction
Train-to-train spacing — computed from the realtime snapshot using both stop-sequence position and origin departure times
What's NOT possible from your current data (and the code fixes it):

Actual arrival/departure timestamps, actual dwell time, actual headway, per-station delay propagation, and sequence modeling with delay data all require TripUpdate entities from the GTFS-RT feeds. Your existing collect_realtime_snapshot() only collects entity.vehicle (VehiclePosition). The new collect_trip_updates() function in the cell fetches entity.trip_update, which contains per-stop predicted arrival/departure times and delay values.
One limitation that can't be worked around:

Scheduled dwell time is always 0 in MTA's static GTFS because they set arrival_time == departure_time for every stop. Actual dwell comes from the TripUpdate data.
The code has two fallback paths for delay propagation: if the TripUpdate feed includes arrival_delay directly, it uses that; otherwise it matches realtime trips to the schedule by parsing origin times and shape suffixes from the trip IDs to compute delay as actual_arrival - scheduled_arrival.

Read the first ~30 lines of all 10 GTFS/static and realtime CSV paths. Listed full headers and representative sample rows. Shell size queries returned no output in this environment; approximate scale is inferred from reader line-count hints where shown (e.g. stop_times is very large). calendar_dates begins with headers only in the excerpt; calendar and agency are small.

The exploration task has finished confirming the structure of all 10 data files. Everything aligns with the analysis and code I already wrote in cell 4 of the notebook. You can go ahead and run that cell — it will compute all the features that are calculable and also collect the TripUpdate data from the live GTFS-RT feeds to fill in the gaps (actual timestamps, actual dwell, delay propagation, and sequence features).





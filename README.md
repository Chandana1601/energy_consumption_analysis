# energy_consumption_analysis
This Python script connects to a MySQL database (infotainment_energy) and analyzes energy usage logs from the energy_logs table. It performs the following steps:  Retrieves energy usage data (CPU, display, audio, connectivity power, and system state) ordered by timestamp. Loads the data into a pandas DataFrame for analysis. 
import mysql.connector
import pandas as pd

# Connect to MySQL
mydb = mysql.connector.connect(
    host='****************',
    user='root',
    password='**********',
    database='infotainment_energy'
)
cursor = mydb.cursor()

# Query: Get energy usage logs
cursor.execute("""
    SELECT
        timestamp,
        cpu_power,
        display_power,
        audio_power,
        connectivity_power,
        system_state
    FROM energy_logs
    ORDER BY timestamp;
""")
rows = cursor.fetchall()

# Load into pandas DataFrame
columns = ['timestamp', 'cpu_power', 'display_power', 'audio_power', 'connectivity_power', 'system_state']
df = pd.DataFrame(rows, columns=columns)

# Feature Engineering: Total power, peak usage, state-based averages
df['total_power'] = df['cpu_power'] + df['display_power'] + df['audio_power'] + df['connectivity_power']
df['peak_usage'] = df['total_power'].rolling(window=10).max()

# Group by system state for average consumption
state_avg = df.groupby('system_state')[['cpu_power', 'display_power', 'audio_power', 'connectivity_power', 'total_power']].mean().reset_index()
state_avg.to_csv('energy_state_averages.csv', index=False)

# Save processed log for Power BI
df.to_csv('energy_consumption_log.csv', index=False)

cursor.close()
mydb.close()

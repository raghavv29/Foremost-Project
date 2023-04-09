import subprocess
import os
import shutil

# Set the path to the test DataStream containing video files
datastream_path = "/path/to/test/DataStream"

# Set the path to the directory for storing the recovered video files
output_dir = "/path/to/output/directory"

# Set the path to the directory for storing partially recovered video files
partial_dir = "/path/to/partial/directory"

# Create the output directory if it doesn't exist
if not os.path.exists(output_dir):
    os.mkdir(output_dir)

# Create the partial directory if it doesn't exist
if not os.path.exists(partial_dir):
    os.mkdir(partial_dir)

# Define the command to run Foremost
foremost_cmd = ["foremost", "-t", "avi,mov,mp4", "-i", datastream_path]

# Run Foremost and capture the output
output = subprocess.check_output(foremost_cmd)

# Extract the list of recovered video files from the output
recovered_files = [line.decode().split(": ")[1].strip() for line in output.split(b"\n") if b"Found" in line]

# Move the fully recovered video files to the output directory
for file_path in recovered_files:
    file_name = os.path.basename(file_path)
    output_path = os.path.join(output_dir, file_name)
    shutil.move(file_path, output_path)

# Search for partially recovered video files
partial_files = []
for root, dirs, files in os.walk(datastream_path):
    for file_name in files:
        if file_name.endswith(".avi") or file_name.endswith(".mov") or file_name.endswith(".mp4"):
            file_path = os.path.join(root, file_name)
            file_size = os.path.getsize(file_path)
            if file_size > 0 and file_size < os.path.getsize(output_path):
                partial_files.append(file_path)

# Move the partially recovered video files to the partial directory
for file_path in partial_files:
    file_name = os.path.basename(file_path)
    partial_path = os.path.join(partial_dir, file_name)
    shutil.move(file_path, partial_path)

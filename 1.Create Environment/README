
### 1. Open Your Terminal

### 2. Clone the GitHub Repository

1. **Find the Repository URL:**
   - Click the green **Code** button.
   - Copy the URL (it might look like `https://github.com/yourusername/fastq-qc-preprocessing.git`).

2. **Clone the Repository:**
   - In your terminal, type the following command (replace the URL with the repository’s URL):

     ```bash
     git clone https://github.com/yourusername/fastq-qc-preprocessing.git
     ```

   - This command downloads a copy of the repository into a new folder on your computer.

---

### 3. Navigate to Your Repository Folder

After cloning, you need to "change directory" to your repository folder.

```bash
cd fastq-QC
```

Now you are in the folder that contains the `environment.yml` file.

---

### 4. Create the Conda Environment

Make sure you have Conda installed (Miniconda or Anaconda). Once in your repository folder, type:

```bash
conda env create -f environment.yml
```

This command tells Conda to create a new environment using the list of tools and packages defined in `environment.yml`. The environment is fastq-qc.

---

### 5. Activate the Environment

After the environment is created, activate it by typing:

```bash
conda activate fastq-qc
```

You should now see that your command prompt has changed (it might show `(fastq-qc)` before your username).

---

### 6. Verify the Installation

Now that your environment is active, you can check if the tools are installed by typing:

```bash
fastqc --version
trimmomatic -version
fastuniq -h
flash --help
```

Each command should show some information about the tool instead of an error message.

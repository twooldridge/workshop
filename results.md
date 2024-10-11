# QC

In your snpArcher `results/cracherodii` folder you should see something that looks like this:

<img width="908" alt="image" src="https://github.com/user-attachments/assets/e350f777-b5f2-4570-b883-fa531daf0700">

The first thing we'll take a look at is the overall QC data located in `.QC/cracherodi_qc.html`. Let's transfer this from the cluster to our local machine, and examine in a browser. **Organizational tip** for server -> local transfers, I like to create a folder in my project directory called something like `transfer`, and put the file I want in that directory. Then, on my local machine, I can just always enter the same command:

```
scp -r ${USERNAME}@${SERVER}:${WORKDIR}/transfer ~/Downloads
```
Where you replace all those variables with your server-specific setup. 

Now, the results!

![image](https://github.com/user-attachments/assets/dd8f9fd5-e8b1-4384-a6ca-c4a7523b9483)

Okay, this dashboard is **extremely** useful for initial exploration of the data and making sure everything looks ok. For example, no weird outliers on the PCA. The reality is that you will likely end up re-running many of these analyses yourself as you continue to explore your data and remove individuals, variants, etc. So think of this as what the name implies: a QC tool. 



# VCFs


Let's quickly inspect the VCFs to see what the differences are between them. We see that the `raw` folder still retains ALL snps and indels produced by the pipeline, but you can see some of the sites have been flagged by quality filters:

```
bgzip -dc cracherodi_raw.vcf.gz | grep -v "##"  | less -S****
```

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/79b84c93-868c-45a1-b2ef-494c05f9d974">

 And those sites have been removed in the `filtered` vcf, which is what we'll use for additional downstream analyses.

<img width="1168" alt="image" src="https://github.com/user-attachments/assets/df5c72a1-da7d-439a-9eb0-37b016b3dfff">

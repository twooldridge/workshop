In your snpArcher results folder you should see something that looks like this:

<img width="908" alt="image" src="https://github.com/user-attachments/assets/e350f777-b5f2-4570-b883-fa531daf0700">

Let's quickly inspect the VCFs to see what the differences are between them. We see that the `raw` folder still retains ALL snps and indels produced by the pipeline, but you can see some of the sites have been flagged by quality filters:

```
bgzip -dc cracherodi_raw.vcf.gz | grep -v "##"  | less -S****
```

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/79b84c93-868c-45a1-b2ef-494c05f9d974">

 And those sites have been removed in the `filtered` vcf, which is what we'll use for additional downstream analyses.

 <img width="1181" alt="image" src="https://github.com/user-attachments/assets/3e965854-21eb-4489-b7d5-00c8eee95ade">

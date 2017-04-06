```
some important arrays that are loaded fromindex files:
buffer_kmer_g: store kmer's kmer part; load from index file unipath_g.kmer
buffer_hash_g: store kmer's hash part; load from index file unipath_g.hash
buffer_off_g:  store kmer's offset on unipath seq; load from index file unipath_g.offset

buffer_seqf: store unipath's offset on unipath seq; load from index file unipath.seqfb
buffer_seq: store unipath seq (concatenate all unipaths' seq); load from index file unipath.seqb
buffer_pp: store unipath's pointer to array buffer_p; load from index file unipath.posp
buffer_p: store every unipath's set of positions on reference; load from index file unipath.pos

buffer_ref_seq: store reference seq; load from index file ref.seq

chr_names and chr_end_n: store every chr name and its lengths; load from index file unipath.chr(非二进制)

index file unipath.size store some values about size of arrays above, which is used to malloc memory for arrays

notice: 其他的没提到的索引文件没有用了 因为现在deBGA版本和最开始的版本索引文件设计不太一样，有些没用索引文件一直都没删，index building现在还是会生成出来，这算是个bug，下个版本更新会解决这个问题

kmer_bit: the kmer that is to be searched
const uint8_t k = 14;
k_r = k_t - k; //k_t is the length of kmer (21-28, default=22)
bit_tran[] is defined in bit_operation.c

seed_kmer = (kmer_bit & bit_tran[k_r]); //get the kmer's kmer part
seed_hash = (kmer_bit >> (k_r << 1));	//get the kmer's hash part


UTkmer:
//binary search a kmer in index, defined in binarys_qsort.h, seed_binary_r == -1 if cannot find this kmer	
int64_t seed_binary_r = binsearch_offset(seed_kmer, buffer_kmer_g, buffer_hash_g[seed_hash + 1] - buffer_hash_g[seed_hash], buffer_hash_g[seed_hash]);

//get this kmer's offset on unipath seq
kmer_pos_uni = buffer_off_g[seed_binary_r];

//binary search on unipath offset array, to get the unipathID that this kmer belongs to, defined in binarys_qsort.h
//seed_id_r is this kmer's UID
seed_id_r = binsearch_interval_unipath(kmer_pos_uni, buffer_seqf, result_seqf);

即通过上述2个二分查找函数可获得kmer所在Uniapth ID (UID)
//get this kmer's offset on its unipath
uni_offset_s_l = kmer_pos_uni - buffer_seqf[seed_id_r];

UTseq:
//this kmer's unipath length
unitig length = buffer_seqf[seed_id_r + 1] - buffer_seqf[seed_id_r];
//get this kmer's unipath seq
for(i = buffer_seqf[seed_id_r]; i < buffer_seqf[seed_id_r + 1]; i++)
A[a++] = buffer_seq[i];
 
UTpos:
//get the this unipath's number of positions on reference
ref_pos_n = buffer_pp[seed_id_r + 1] - buffer_pp[seed_id_r];

//get this UID's set of positions
uint32_t* pos_tmp_p = buffer_p + buffer_pp[UID];
for(id_i = 0; id_i < pos_n; id_i++)
A[a++] = pos_tmp_p[id_i]
```



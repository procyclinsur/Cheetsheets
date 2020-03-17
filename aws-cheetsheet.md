# AWS Cheetsheet

# Start a dialog for deleting S3 buckets quickly.
function clean_s3 () {
    echo -e "Gathering Bucket List....\n"
    bucket_list=()
    for i in $(aws s3api list-buckets --query 'Buckets[*].Name'); do 
        REGION=$(aws s3api get-bucket-location --bucket $i)
        bucket_list+=("$i $REGION")
    done
    for line in "${bucket_list[@]}"; do
        echo -e "BUCKET: ${line%" "*}"
        echo -e "REGION: ${line#*" "}"
        read -p "delete[yes/no](default:no):" ans < /dev/tty
        if [ "$ans" == "yes" ]; then
            aws s3 rm s3://${line%" "*} --recursive
            aws s3api delete-bucket --bucket ${line%" "*}
            echo -e "Bucket ${line%" "*} \033[0;32mDELETED\033[0m\n"
        else
            echo -e "Bucket ${line%" "*} PRESERVED\n"
        fi 
    done < <(echo -e $bucket_list)
}

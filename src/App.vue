<template>
  <div id="app">
    <b-form @submit="onSubmit">
      <b-form-group
          id="input-group-1"
          label="Tag:"
          label-for="input-1"
      >
        <b-form-input
            id="input-1"
            v-model="tag"
            type="text"
            placeholder="Enter email"
            required
        ></b-form-input>
      </b-form-group>

      <b-button type="submit" variant="primary">Submit</b-button>
    </b-form>

    <div v-for="(item, index) in final" :key="index">
      <b-button v-b-toggle="'item'+item.question_id" variant="primary">{{ item.title }}
        creation Date: {{convertToHumanDate(item.creation_date)}}
        vote: {{item.score}} </b-button>
      <b-collapse :id="'item'+item.question_id" class="mt-2">
        <b-card>
          <p>body: {{ item.body }}</p>
          <p>answers: {{ item.answersContent }}</p>
          <p>comments: {{ item.commentsContent }}</p>
          <p>creation date: {{convertToHumanDate(item.creation_date)}}</p>
          <p>vote: {{item.score}}</p>
        </b-card>
      </b-collapse>
    </div>
    total network time not including logic {{this.totalNetworkTime}} milliseconds
  </div>
</template>

<script>
// eslint-disable-next-line no-unused-vars
import axios from "axios"
// eslint-disable-next-line no-unused-vars
import dayjs from "dayjs"
import _ from "lodash"


export default {
  data() {
    return {
      tag: "",
      show: true,
      newestParsed: [],
      newestResponse: [],
      mostVotedRes: [],
      final: [],
      totalNetworkTime: 0,
    }
  },
  methods: {
    convertToHumanDate(unixTime) {
      return dayjs.unix(unixTime).format("YYYY-MM-DD");
    },

    async onSubmit(event) {
      this.totalNetworkTime = 0;
      event.preventDefault();
      this.newestParsed = [];
      this.newestParsed = [];

      let startDate = dayjs();
      let endDate = dayjs().subtract(7, "day");
      let todate = startDate.format("YYYY-MM-DD")
      let fromdate = endDate.format("YYYY-MM-DD")
      let startTime = dayjs().valueOf();
      this.newestResponse = await axios.get("https://api.stackexchange.com/2.2/questions/",  {params: {filter:"withbody", site:"stackoverflow", tagged:this.tag, sort:"creation", order:"desc"}})
   this.mostVotedRes = await axios.get("https://api.stackexchange.com/2.2/questions/",  {params: {filter:"withbody", site:"stackoverflow", tagged:this.tag, sort:"votes", order:"desc", fromdate:fromdate, todate:todate}})
      let endTime = dayjs().valueOf();
      this.totalNetworkTime += (endTime - startTime);


      for(let i=0; i<10 && i< this.newestResponse.data.items.length; i++){
        this.newestParsed.push(this.newestResponse.data.items[i]);
      }

      for(let i=0; i<10 && i< this.mostVotedRes.data.items.length; i++){
        this.newestParsed.push(this.mostVotedRes.data.items[i]);
      }

      this.newestParsed = _.orderBy(this.newestParsed, ["creation_date"], ["desc"])


        let questionIds = this.newestParsed.map(q => q.question_id);
        let idMessage = "";
        for(let i=0; i<questionIds.length; i++){
          idMessage = idMessage + questionIds[i];
          if(i !== questionIds.length-1){
            idMessage = idMessage + ";"
          }
        }
      startTime = dayjs().valueOf();
    let answersResponse = await axios.get(`https://api.stackexchange.com/2.2/questions/${idMessage}/answers?site=stackoverflow&filter=withbody`)
    let commentsResponse = await axios.get(`https://api.stackexchange.com/2.2/questions/${idMessage}/comments?site=stackoverflow&filter=withbody`)
    endTime = dayjs().valueOf();

    this.totalNetworkTime += (endTime - startTime);

      commentsResponse = commentsResponse.data.items.map(c => ({body:c.body, question_id:c.post_id}) )
      answersResponse = answersResponse.data.items.map(c => ({body:c.body, question_id:c.question_id}) )


      answersResponse = _.groupBy(answersResponse, "question_id");
      commentsResponse =_.groupBy(commentsResponse, "question_id");


      for(let q of this.newestParsed) {
        if (answersResponse[q.question_id]) {
          q.answersContent = answersResponse[q.question_id].map(a => a.body);
        } else {
          q.answersContent = []
        }
        if (commentsResponse[q.question_id]) {
          q.commentsContent = commentsResponse[q.question_id].map(a => a.body);
        } else {
          q.commentsContent = []
        }
      }

      this.final = this.newestParsed;
      },
  }
}
</script>